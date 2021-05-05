---
layout: post
title:  "  SOLID design principles in Terraform"
categories: Architecture
---



### Introduction

In this article I would like to discuss the ability to use SOLID principles for terraform language. I assume that each principle may be adapted for terraform and I will provide examples for each one.


### SOLID

Upon [wikipedia](https://en.wikipedia.org/wiki/SOLID) SOLID is
```

mnemonic acronym for five design principles intended to make software designs more understandable, flexible, and maintainable
```

With that I want to note that I didn’t find any scientific evidence that those design principles really will make your code understandable or flexible.  And still it’s usage in OOP is considered to use them as best practices. In this article I will try to adopt them for declarative programming languages such as terraform.  My understanding of those principles is based on next articles:
http://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html
https://www.digitalocean.com/community/conceptual_articles/s-o-l-i-d-the-first-five-principles-of-object-oriented-design

And mentioned above wikipedia page. I will recommend to review mentioned pages before proceeding with terraform examples. Also I would say that I will focus mostly on Liskov Substitution principle from Uncle Bob. And as he mentioned in his article, DigitalOcean’s version of this design principle is wrong, because it is based on inheritance.

But before we begin describing each principle we have to define  what is  `class` and  `interface` notions in terraform world. There are definitely a similar concept for `class` which is a [terraform module](https://www.terraform.io/docs/language/modules/develop/index.html) because of its ability to create lightweight abstractions and be recallable multiple time (like creating multiple objects).

Interface(or [protocol](https://en.wikipedia.org/wiki/Protocol_(object-oriented_programming))) is a harder example.Up to wikipedia :
```
Protocol is a definition of methods and values which the objects agree upon, in order to cooperate, as part of an API .
```
 I think that [terraform input variables](https://www.terraform.io/docs/language/values/variables.html) are a most suitable notion for interface/protocol definition. So further I will understand terraforms module parameters as an interface.


Keeping above in mind let’s get back to SOLID design principles.

#### Single-responsibility principle

```
A class should only have a single responsibility, that is, only changes to one part of the software's specification should be able to affect the specification of the class.
```
For me it is mostly about choosing proper abstraction for your modules. For example let’s review the next case. You have some terraform code that should create a kubernetes cluster and  install some software in there.  For this scenario you will create 2 modules: one to spin up the k8s cluster, and second to install software.

#### Dependency inversion interface

```
Depend in the direction of abstraction. High level modules should not depend upon low level details.

```
Continuing with the previous example let’s pretend that we want to deploy minio application with helm.  Helm’s part may be defined next way:

```
provider "helm" {
  kubernetes {
    load_config_file       = false
    host                   =  "https://${google_container_cluster.cluster.endpoint}”
    token                  = var.kubernetes_access_token
    cluster_ca_certificate = google_container_cluster.cluster.master_auth[0].cluster_ca_certificate
  }
}

resource "helm_release" "minio" {
  name       = "minio"
  chart      = "minio"
  repository = "https://helm.min.io/"
  namespace         = “minio”
}
```
And gke cluster:
```
provider "google" {
  project     = "my-project-id"
}
resource "google_container_cluster" "cluster" {
  name     = "my-gke-cluster"
  location = "us-central1"
  initial_node_count       = 1
}
```

Managing all resources in the same module will be a violation  of dependency inversion interface principle because  you will need to know low level details( google project ) to change minio namespace(higher level abstraction). So it is better to separate the helm chart and gke provisioning in separate modules.

### Open-Closed principle
```
A Module should be open for extension but closed for modification.
```

A [feature toggle design pattern](https://build5nines.com/terraform-feature-flags-environment-toggle-design-patterns/) will suit here well.  Our module should provide base functionality(for example create gke cluster). And if in future we will need to add some compute firewall rule we can implement it as a feature toggle in our module and set it true only for one  particular cluster.

### The Interface Segregation Principle
```
A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.

```

In the same example as for open-closed principle, if we always will be creating a gke cluster with additional firewall rule (which may require ip address from input variable) then client will be forced to provide ip for firewall rule all the time, even if this particular client do not need such rule at all.

### The Liskov Substitution Principle
```
A program that uses an interface must not be confused by an implementation of that interface.

```

???
