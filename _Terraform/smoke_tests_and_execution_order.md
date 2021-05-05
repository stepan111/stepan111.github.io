
---
layout: post
title:  "Smoke test and blocking deployments"
categories: Terraform
---

 Terrafrom  support `depends_on` for modules since 0.14. But when you have mixed push/pull deployment process (like provisioning k8s resources with terraform - terraform push  yaml and k8s apply those yaml during pull model ) you may need some kind of validation in your module to make it's execution blocking.

 Other case may be a smoke test when you want to validate your service running after deployment.

I found that running a  blocking smoke test for each component it is the best testing strategy for Infrastructure as code. With such approach you gain fastest feedback on component failures and in conjunction with `depends_on` modules you won't need to search root cause in dependent components.

Here are my example of such *blocking* module:
```
resource "helm_release" "elasticsearch" {
  name   = local.esValues.esName
  chart  = "${path.module}/elasticsearch"
  atomic = true
  values = [
    yamlencode(local.esValues)
  ]
}

resource "null_resource" "ansible_validate_resources" {
  triggers = {
    playbook_contents = md5(file("${path.module}/../../ansible/validate.yml"))
    helm_release      = helm_release.elasticsearch.id
  }
  provisioner "local-exec" {
    command = <<COMMAND
    cd ${path.module}/../../ansible &&\
    ansible-playbook validate.yml \
      -e 'ansible_python_interpreter=/usr/bin/python3'
COMMAND

    environment = merge(local.ansibleEnvs, {
      name       = local.esValues.esName
      kind       = "Elasticsearch"
      apiVersion = "elasticsearch.k8s.elastic.co/v1"
    })
  }
  depends_on = [
    helm_release.elasticsearch
  ]
}

```
