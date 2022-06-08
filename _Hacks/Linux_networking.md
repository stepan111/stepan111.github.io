---
layout: post
title:  "Linux Networking"
categories: Hacks
---

> In Linux, by default, packets are considered individually for routing purposes. Thus, all the routing algorithm determines where to send a packet based on that packet itself, without taking into consideration that the packet may be a response packet of sorts.
> In a typical setup, this means that all outgoing traffic is going out over one interface, say, eth0 even if the incoming packet was sent to interface eth1.

    <div style="text-align: right"> [softpanorama](http://www.softpanorama.org/Net/Internet_layer/Routing/martian_source.shtml)</div>




For example like this:
```
+------------+        /
|            |       |
+-------------+ Gateway 1  +-------
|             | 10.1.1.1   |     /
+------+-------+     +------------+    |
|     eth0     |                      /
|   10.1.1.2   |                      |
|              |                      |
| DOCKER HOST  |                      |
|              |                      | Internet
|   docker0    |                      |
|   (bridge)   |                      |
|  172.17.42.1 |                      |
|              |                      |
|     eth1     |                      |
|  192.168.1.2 |                      \
+------+-------+     +------------+    |
|             |            |     \
+-------------+ Gateway 2  +-------
| 192.168.1.1|       |

```
When you have multiple interfaces on your Linux box and each interface connected to internet you better remember about policy routing and rp_filter.
This serverfault [post](https://serverfault.com/questions/618857/list-all-route-tables) answer perfectly but for me it takes hours of downtime when you not doing it each day.

Here are some useful commands for network troubleshooting:

```
ip rule list

ip route show table all | grep -Po 'table \K[^\s]+' | sort -u

tcpdump -nni eth0 port 80

docker run -it --rm --net container:<container-id>  corfr/tcpdump -i eth0 port 443

sysctl -a | grep rp_filter | grep -v ipv6

# sysctl net.ipv4.conf.ens5.rp_filter=2

# sysctl net.ipv4.conf.all.rp_filter=2

dmesg -Tw | grep -i martian

conntrack -L

watch iptables -nvL DOCKER -t nat

watch iptables -nvL DOCKER

```

Also remember regarding  [iptables packet processing flow](https://uk.wikipedia.org/wiki/Iptables#/media/%D0%A4%D0%B0%D0%B9%D0%BB:Netfilter-packet-flow.svg)
