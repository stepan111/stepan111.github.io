
---
layout: post
title:  "Use locals to get clear syntax"
categories: Terraform
---

This topic may be related to refactoring and some kind of 'smells' in terraform.
When you trying to implement some logical choices or process something in terraform you will get not really nice multiline expression that won't be readable inside your resource block.
I suppose that [locals](https://www.terraform.io/docs/language/values/locals.html)  are best choice to resolve such issues. Let's proceed with examples.

Feature toggles:
```
locals {
  initialProvisioningToggle = var.Settings.Nested.Structure["Here"].do_initial_provisioning ? 1 : 0
}

resource "null_resource" "initial_provisioning_playbook" {
  count = local.initialProvisioningToggle

  ...
}
```
Nested structure processing. In this key we iterate over list of maps `namepaces`, where some maps may have key `devAccess` :
```
locals{
  namespaceDevAccess = toset(flatten([
    for namespace in var.Settings.cluster.namespaces : [
      namespace.name
    ] if can(namespace.devAccess)
  ]))
}

resource "kubernetes_role_binding" "developers_groups_binding" {
  for_each = local.namespaceDevAccess
  metadata {
    name      = "developers-access"
    namespace = each.key
  }

 ...
}

```

But in more complex cases it is better to use maps then lists. For example if we want to create GKE nodepools with `for_each`, we may need config.auto.tfvars.json like this:
```
"Settings":

...

"additionalNodepools": {
  "db": { "taintWorkloadsType": "data", "machineType": "e2-standard-4", "minNodes": 1,"maxNodes": 1, "preemptible": true},
  "web": { "taintWorkloadsType": "web", "machineType": "e2-standard-4", "minNodes": 1,"maxNodes": 1, "preemptible": true}
}
```

And terraform  config:

```
locals{
  additionalNodepools = var.Settings.Nested["reference"].to.additionalNodepools
}


resource "google_container_node_pool" "nodepool_additional" {
  for_each = local.additionalNodepools
  lifecycle {
    ignore_changes = all
  }

  provider = google-beta
  name     = "${var.Cluster}-${var.Color}-${each.key}"
  location = var.Settings.clusters[var.Cluster].region
  cluster  = google_container_cluster.cluster.name

  autoscaling {
    min_node_count = each.value.minNodes
    max_node_count = each.value.maxNodes
  }
  node_count = each.value.minNodes

  node_config {
    preemptible     = each.value.preemptible
    machine_type    = each.value.machineType
    tags            = [var.Cluster,"${var.Cluster}-${var.Color}","${var.Cluster}-${var.Color}-${each.key}"]
    service_account = google_service_account.gke_sa.unique_id
    metadata = {
      disable-legacy-endpoints = "true"
    }
    oauth_scopes = [
      "https://www.googleapis.com/auth/logging.write",
      "https://www.googleapis.com/auth/monitoring",
    ]
    labels = {
      "workloadsType" = each.value.taintWorkloadsType
    }
    taint {
      key    = "workloadsType"
      value  = each.value.taintWorkloadsType
      effect = "NO_SCHEDULE"
    }
  }
  management {
    auto_repair  = "true"
    auto_upgrade = "true"
  }

  timeouts {
    create = "60m"
    update = "20m"
  }
}


```
