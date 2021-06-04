---
layout: post
title:  "[ WIP ] Refactoring for declarative languages"
categories: Architecture
---


Here are some clean code principles that can be used for declarative languages:

* Naming convention([terraform](https://github.com/terraform-linters/tflint/blob/v0.16.2/docs/rules/terraform_naming_convention.md))
* DRY - no duplication
* KISS - minimize dependecies
* YAGNI - Minimizes the number of entities
* SOLID - choose abstractions and interfaces properly
* Automate  your linter usage



When you creating module with arguments always try to set default values for that arguments!


As for me the most complex is to keep configurations DRY. Here are some useful features on this topic:
* terraform modules
* terragrunt
* module params

When you develop your infrastructure from scratch you may need next components(IaC modules/code structure):
* Network Layer - this will contain all automation to create and configure your network resources(VPC,subnets,jump-hosts,VPN,etc.). Network layer is essential and always be provisioned first.
* Secret storage - You want to have secret storage which will keep at least persistent secrets. For example hashicorp vault.
* Shared Config - you will need a shared configuration storage/discovery system for your infrastructure code. Multiple modules/projects will refer it during execution. Simple example - JSON config included in all projects. Consul or etcd will be more advanced solution. You may choose any to your needs.
* Compute resources - this is main modules which will contain your infrastructure services( Databases, miscellaneous apps, etc. ).  I suppose this will be modules that you change most.
* Persistant resources - modules/projects that define resources refered by compute (storage for backups, policies and service accounts, etc.)

Here are my thoughts on module composition for IaC. Initially I was thinking that having one large json config would be more convenient than defining vars in each module. It may help to share data between modules. And for some cases it worked. Now I want to decide what data should be kept in a shared json and what should be defined in a module.

Shared:
* IPAM. All addresses and related network information. Generally you want to  keep network provisioning  separate from component provisioning modules. But all those modules need IPAM and DNS related settings. Also it is useful when you developing firewall rules for each component separately.
* Versions of components. Keeping versions of all components in one place is useful. Generally it will help to solve integration problems and/or compare environments
* Namespace settings. I am keeping specific settings for k8s namespaces like vault roles assignment and additional credentials to provision in one place and multiple terraform/ansible modules refer to this data.
* Persistent resources.  All resources that should be kept after k8s cluster deletion(Service Accounts, S3 buckets, etc). This data is also referred by multiple modules.
* Active cluster. I am using active/passive k8s clusters and keeping data regarding what cluster is active right now in a shared config file is crucial.

Now lets focus which settings will overload shared configuration and better to keep in module params:
* Feature toggles. Initially I thought that it would better describe the component. In fact it just creates additional lines that you want to scroll each time you are working with a shared configuration file. Now i am going to move this setting into modules with default settings.

Also in case of blue/green k8s clusters you definitely want to have one terraform project for provisioning both envs with help of `terragrunt`. Similar is relevant for STAGE to PROD changes transition.
