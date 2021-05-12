---
layout: post
title:  "Push or pull"
categories: Architecture
---

Some points here:
* Terraform is perform a push model (same as ansible)
* You better use terraform when you have dependecies between componenets
* When you have loosely coupled components it is better to use pull model (like fluxcd or argocd from k8s world)
* when you trigger your terraform regularly from jenkins or gitlab - you developed the pull model. You gain proper dependency management in your pull deployment. Negative part that you gain time consuming deployment and often unpredictable (nobody will review `terraform plan`).
