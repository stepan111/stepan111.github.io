---
layout: post
title:  "Kind cluster with valid certificates"
categories: Hacks
---
> For proper local testing and more prod-like environment we need our cluster to have valid DNS records and certificates. For this purpose I am going to use MetalLB,Nginx Ingress, cert-manager and external DNS pointing to CloudFlare site.

## Steps:

### 1. Put kind config:
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker

```
and create cluster:
```
sudo kind create cluster --config ./kind.yaml --name cluster

```
There are 2 types of networking in podman: rootless and rootfull. The first one is used for regualr users and will not allow you to communicate from host to container.
Here are (details)[https://www.redhat.com/sysadmin/container-networking-podman]. That's why we need to create cluster with sudo and copy credentials to local user.


### 2. Create `provider.tf` : 
```
provider "helm" {
  kubernetes {
    config_context = "kind-cluster"
    config_path    = "~/.kube/config"
  }
}

provider "kubernetes" {
  config_context = "kind-cluster"
  config_path    = "~/.kube/config"
}

provider "kubectl" {
  config_context = "kind-cluster"
  config_path    = "~/.kube/config"
}


terraform {

  required_providers {
    kubectl = {
      source  = "registry.terraform.io/gavinbunney/kubectl"
      version = ">= 1.10.0"
    }
  }
}

```
### 3. Create `metal-lb.tf`:
```
resource "helm_release" "metal_lb" {
  name              = "metallb"
  chart             = "metallb"
  repository        = "https://metallb.github.io/metallb"
  create_namespace  = "true"
  namespace         = "metallb"
  dependency_update = "true"
}

locals {
  # Check address pool with next command:
  # docker network inspect kind | jq '.[0].plugins[0].ipam.ranges'
  # We need part of nodes address that will not overlap with any assigned address
  IPAddressPool = <<EOF
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb
spec:
  addresses:
  - 10.89.0.128/25
EOF

  L2Advertisement = <<EOF
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb
spec:
  ipAddressPools:
  - first-pool
EOF
}

resource "kubectl_manifest" "address_pool" {
  yaml_body = local.IPAddressPool
  depends_on = [
    helm_release.metal_lb
  ]
}
resource "kubectl_manifest" "l2_advertisement" {
  yaml_body = local.L2Advertisement
  depends_on = [
    helm_release.metal_lb
  ]
}

```
### 4. Create `cert-manager.tf`
```
variable "CfApiToken" {
}

resource "helm_release" "cert_manager" {
  name       = "cert-manager"
  repository = "https://charts.jetstack.io"
  chart      = "cert-manager"
  #version    = 
  namespace        = "cert-manager"
  create_namespace = "true"
  set {
    name  = "installCRDs"
    value = "true"
  }
  // Configure cert-manager to cleanup secrets on delete
  // https://github.com/jetstack/cert-manager/pull/819
  set {
    name  = "extraArgs[0]"
    value = "--enable-certificate-owner-ref=true"
  }
}

resource "kubernetes_secret" "cf_api_token" {
  metadata {
    name      = "cf-api-token"
    namespace = "cert-manager"
  }

  data = {
    token = var.CfApiToken
  }

  type = "Opaque"
  depends_on = [
    helm_release.cert_manager
  ]
}


locals {
  clusterIssuer = <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
 name: letsencrypt-prod
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
         name: letsencrypt-prod
    solvers:
    - selector:
        dnsZones:
          - "local-k8s.pp.ua"
      dns01:
          cloudflare:
            apiTokenSecretRef:
              name: cf-api-token
              key: token

EOF
}

resource "kubectl_manifest" "cluster_issuer" {
  yaml_body = local.clusterIssuer
  depends_on = [
    helm_release.cert_manager
  ]
}

```
### 5. Create `external-dns.tf`:
```
locals {
  dnsValues = <<EOF
args:
 - --source=service
 - --source=ingress
 - --domain-filter=local-k8s.pp.ua
 - --provider=cloudflare
provider: "cloudflare"
cloudflare:
  secretName: ${kubernetes_secret.cf_api_token_2.metadata.0.name}
  proxied: false
EOF
}

resource "helm_release" "external-dns" {
  name              = "external-dns"
  chart             = "external-dns"
  repository        = "https://charts.bitnami.com/bitnami"
  create_namespace  = "true"
  namespace         = kubernetes_namespace.dns.metadata.0.name
  dependency_update = "true"
  values = [
    local.dnsValues
  ]

}

resource "kubernetes_namespace" "dns" {
  metadata {
    name = "external-dns"
  }
}



resource "kubernetes_secret" "cf_api_token_2" {
  metadata {
    name      = "cf-api-token"
    namespace = kubernetes_namespace.dns.metadata.0.name
  }

  data = {
    cloudflare_api_token = var.CfApiToken
  }

  type = "Opaque"

}
```
### 6. And `ingress.tf`:
```

resource "helm_release" "ingress" {
  name              = "ingress-nginx"
  chart             = "ingress-nginx"
  repository        = "https://kubernetes.github.io/ingress-nginx"
  create_namespace  = "true"
  namespace         = "ingress"
  dependency_update = "true"
}


```

### 7. Apply terraform:
```
export TF_VAR_CfApiToken=<CloudFlare Api token for DNS zone management>
alias tf=terraform
tf apply

```

### 8 .Test install
```
alias ksn='kubectl config set-context --current --namespace '
ksn default
k run nginx --image=nginx
k expose po nginx --port=80
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: letsencrypt-prod
  name: nginx-test
  namespace: default
spec:
  rules:
  - host: nginx-test.local-k8s.pp.ua
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: nginx
            port:
              number: 80
  # This section is only required if TLS is to be enabled for the Ingress
  tls:
    - hosts:
      - nginx-test.local-k8s.pp.ua
      secretName: nginx-tls
EOF
```