---
layout: post
title:  "Awscli hacks"
categories: Hacks
---



### describe instances queries
```
ec2 describe-instances --filters  --query 'Reservations[].Instances[][Placement.AvailabilityZone, State.Name, Tags[?Key==`Name`].Value ]'

```

```
--query 'Reservations[].Instances[][Placement.AvailabilityZone, SubnetId, State.Name, InstanceId, NetworkInterfaces[?Status ==`in-use`].[PrivateIpAddresses[?Primary==`true`].PrivateIpAddress], Tags[?Key==`Name`].Value]

```


```
aws ec2 describe-instances --filters Name=tag:ExternalName,Values=cops-test-flex.cloud.modeln.com Name=instance-state-name,Values=running,stopped --region us-east-1 --profile profile-name --output json --query 'Reservations[*].Instances[].{ID:InstanceId,NAME:Tags[?Key==`Name`]|[0].Value}' 
```
```
aws ec2 describe-instances --filters Name=tag:ExternalName,Values=gilead.cloud.modeln.com Name=instance-state-name,Values=running,stopped --region us-east-1 --profile profile-name --output json --query '{IPS:Reservations[*].Instances[*].NetworkInterfaces[?Status ==`in-use`][PrivateIpAddress][][][]}'
```
