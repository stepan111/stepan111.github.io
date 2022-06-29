---
layout: post
title:  "Deploy ArgoCD with vault plugin to local Kind cluster with terraform"
categories: Hacks
---

## Steps:
### 0. Create cluster as described [here](/Terraform/Create_kind_cluster.html) and deploy vault same as [here](/Terraform/vault_dev_mode.html)

### 1. Create `argo.tf` with next content:
```

locals {
  argoCdValues = <<EOF
server:
  extraArgs:
  - "--insecure"
  config:
    users.anonymous.enabled: "true"
    application.instanceLabelKey: argocd.argoproj.io/instance
    repositories: |
      - type: git
        url: https://github.com/stepan111/4key-dashboard.git
    configManagementPlugins: |
      - name: argocd-vault-plugin
        generate:
          command: ["argocd-vault-plugin"]
          args: ["generate", "./"]
  rbacConfigCreate: false


dex:
  enabled: false
repoServer:
  env:
  - name: AVP_TYPE
    value: vault
  - name: VAULT_ADDR
    value: http://vault.vault:8200
  - name: AVP_AUTH_TYPE
    value: k8s
  - name: AVP_K8S_ROLE
    value: allow-all
  initContainers:
      - name: download-tools
        image: alpine:3.8
        command: [sh, -c]
        volumeMounts:
        - mountPath: /custom-tools
          name: custom-tools

        # Don't forget to update this to whatever the stable release version is
        # Note the lack of the `v` prefix unlike the git tag
        env:
          - name: AVP_VERSION
            value: "1.7.0"
        args:
          - >-
            wget -O argocd-vault-plugin
            https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${"$"}{AVP_VERSION}/argocd-vault-plugin_${"$"}{AVP_VERSION}_linux_amd64 &&
            chmod +x argocd-vault-plugin &&
            mv argocd-vault-plugin /custom-tools/

  volumeMounts:
   - mountPath: /usr/local/bin/argocd-vault-plugin
     name: custom-tools
     subPath: argocd-vault-plugin
  volumes:
  - name: custom-tools
    emptyDir: {}

EOF
}

resource "helm_release" "argocd" {
  name              = "argocd"
  chart             = "argo-cd"
  repository        = "https://argoproj.github.io/argo-helm"
  namespace         = "argo"
  dependency_update = "true"
  create_namespace  = "true"

  values = [
    local.argoCdValues
  ]

}

resource "kubernetes_config_map" "rbac" {
  metadata {
    name      = "argocd-rbac-cm"
    namespace = "argo"
  }

  data = {
    "policy.default" = "role:admin"

  }
  depends_on = [
    helm_release.argocd
  ]

}

```
### 2. Apply changes:
```
tf apply

```

### 3. Create sample app file `argo-app.yaml`:
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: secret-test
  namespace: argo
spec:
  destination:
    server: https://kubernetes.default.svc
  project: default
  source:
    path: kustomize/vault
    repoURL: https://github.com/stepan111/4key-dashboard.git
    targetRevision: HEAD
    plugin:
      name: argocd-vault-plugin

```
### 4. Deploy sample app
```
alias k=kubectl
 k apply -f ./argo-app.yaml
```