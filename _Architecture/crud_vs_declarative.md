---
layout: post
title:  "[ WIP ] CRUD vs Declarative in infrastructure"
categories: Architecture
---




CRUD process such as backup restore not allow to upgrade.
Declarative will set your environment to desired state - but those api's harder to develop.
In the end, all of those related to processes/people in your company. IF you want to develop and do GitOPS - Declarative is right choice.
If you have separate "upgrade" cycle - do backup/restore
