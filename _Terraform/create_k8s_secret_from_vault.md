---
layout: post
title:  "Examples to create k8s secret from Hasicorp Vault"
categories: Terraform
---

Here are example how to create registry secret:

```
data "vault_generic_secret" "harbor_token" {
  path = "secret/goharbor/docker_login"
}



resource "kubernetes_secret" "harbor" {
  metadata {
    name      = "harbor"
    namespace = local.ns
  }

  data = {
    ".dockerconfigjson" = <<DOCKER
{
  "auths": {
    "harbor.infra.digaweb.com": {
     "username": "${data.vault_generic_secret.harbor_token.data.user}",
     "password": "${data.vault_generic_secret.harbor_token.data.password}"
     }
    }
  }
DOCKER
  }

  type       = "kubernetes.io/dockerconfigjson"
  depends_on = [kubernetes_namespace.ns]
}

```
And a regular secret:
```
data "vault_generic_secret" "token" {
  path = "secret/token"
}

resource "kubernetes_secret" "token" {
  metadata {
    name      = "token"
    namespace = local.ns
  }

  data = {
    "token" = data.vault_generic_secret.gitlab_token.data.token
  }

}


```
