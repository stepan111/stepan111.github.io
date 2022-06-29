---
layout: post
title:  "Deploy Vault in Dev mode to local Kind cluster with terraform"
categories: Hacks
---




## Steps:
### 0. Create cluster as described [here](/Terraform/Create_kind_cluster.html)
### 1. Create `vault.tf`:
```

locals {
  vaultValues = <<EOF
global:
      enabled: true

injector:
  enabled: false

server:
  dev:
    enabled: true
    devRootToken: "root"
  standalone:
    enabled: true

EOF
}

resource "helm_release" "vault" {
  name       = "vault"
  chart      = "vault"
  repository = "https://helm.releases.hashicorp.com"
  #version           = "0.1.0"
  namespace         = "vault"
  dependency_update = "true"
  create_namespace  = "true"
  values = [
    local.vaultValues
  ]
}

# Create sa for k8s auth. There are issue in k8s >1.24
# https://github.com/hashicorp/terraform-provider-kubernetes/issues/1724

resource "kubernetes_secret" "vault" {
  metadata {
    name      = "vault"
    namespace = "kube-system"
    annotations = {
      "kubernetes.io/service-account.name"      = "vault"
      "kubernetes.io/service-account.namespace" = "kube-system"
    }
  }
  type = "kubernetes.io/service-account-token"
}


resource "kubernetes_manifest" "service_account" {
  manifest = {
    "apiVersion" = "v1"
    "kind"       = "ServiceAccount"
    "metadata" = {
      "namespace" = "kube-system"
      "name"      = "vault"
    }

    "automountServiceAccountToken" = true
  }
  depends_on = [
    kubernetes_secret.vault
  ]
}

resource "kubernetes_cluster_role_binding" "vault" {
  metadata {
    name = "vault-auth"
  }
  role_ref {
    api_group = "rbac.authorization.k8s.io"
    kind      = "ClusterRole"
    name      = "system:auth-delegator"
  }
  subject {
    kind      = "ServiceAccount"
    name      = "vault"
    namespace = "kube-system"
  }
}


```

### 2. Apply terraform and port-forward service with kubefwd:
```
alias tf=terraform
tf init
tf apply
sudo kubefwd svc -c ~/.kube/config -n vault
```

### 3. Create new file `vault-config/provider.tf` with next content :
```
provider "vault" {
  address = "https://vault.vault:8200"
  token   = "root"
}

provider "kubernetes" {
  config_context = "kind-cluster"
  config_path    = "~/.kube/config"
}

```
### 4. Create new file `vault-config/k8s-auth.tf` with next content :
```

data "kubernetes_secret" "vault" {
  metadata {
    name      = "vault"
    namespace = "kube-system"
  }

}

resource "vault_auth_backend" "kubernetes" {
  type = "kubernetes"
}

resource "vault_kubernetes_auth_backend_config" "kubernetes" {
  backend            = vault_auth_backend.kubernetes.path
  kubernetes_host    = "https://kubernetes.default.svc"
  kubernetes_ca_cert = lookup(data.kubernetes_secret.vault.data, "ca.crt")
  token_reviewer_jwt = lookup(data.kubernetes_secret.vault.data, "token")
  #   issuer                 = "api"
  disable_iss_validation = "true"
}


resource "vault_policy" "secrets" {
  name = "secrets-access"

  policy = <<EOT
path "secret/*" {
  capabilities = ["read","list","update","create","delete"]
}
EOT
}


resource "vault_kubernetes_auth_backend_role" "default" {
  backend                          = vault_auth_backend.kubernetes.path
  role_name                        = "allow-all"
  bound_service_account_names      = ["*"]
  bound_service_account_namespaces = ["*"]
  token_ttl                        = 3600
  token_policies                   = ["default", vault_policy.secrets.name]
  audience                         = "vault"
}


```
### 5. apply terraform:
```
cd vault-config
tf init
tf apply
```