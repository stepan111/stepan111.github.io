---
layout: post
title:  "Managing secrets with automation tools"
categories: Hacks
---


There are few rules to manage secrets secure:
* Do not store decrypted secrets on disk
* Do not commit secrets in git


here are some hacks regarding this topics:
* Create k8s secret from vault with terraform
* Refer secret with helm - do not put them in values.yaml
* Refer secret in pod with vault-injector annotations
* Refer secret with ECK CRDs
* .dockerconfigjson secrets
* Copy secret between namespaces with terraform
* Pass secret from terraform to ansible as environment variable
