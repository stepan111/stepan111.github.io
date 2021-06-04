---
layout: post
title:  " Well-known terms "
categories: Architecture
---


### Split brain

 It indicates data or availability inconsistencies originating from the maintenance of two separate data sets with overlap in scope, either because of servers in a network design, or a failure condition based on servers not communicating and synchronizing their data to each other. This last case is also commonly referred to as a network partition.

### GC death spiral

Increased rate of garbage collection (GC) in Java, resulting in increased CPU usage. A vicious cycle can occur in this scenario: less CPU is available, resulting in slower requests, resulting in increased RAM usage, resulting in more GC, resulting in even lower availability of CPU. This is known colloquially as the “GC death spiral.” [Source](https://sre.google/sre-book/addressing-cascading-failures/#increased-rate-of-garbage-collection-gc-in-java-resulting-in-increased-cpu-usage)


### Cache misses

A cache miss is an event in which a system or application makes a request to retrieve data from a cache, but that specific data is not currently in cache memory.

### dependency hell
Dependency hell is a colloquial term for the frustration of some software users who have installed software packages which have dependencies on specific versions of other software packages. [Wiki](https://en.wikipedia.org/wiki/Dependency_hell)

### Callback Hell
This is a big issue caused by coding with complex nested callbacks. Here, each and every callback takes an argument that is a result of the previous callbacks. In this manner, The code structure looks like a pyramid, making it difficult to read and maintain. Also, if there is an error in one function, then all other functions get affected. [Source](https://www.geeksforgeeks.org/what-is-callback-hell-in-node-js/)


### VM sprawl

Virtualization sprawl or VM sprawl is defined as a large amount of virtual machines on your network without the proper IT management or control. For example, you may have multiple departments that own servers begin creating virtual machines without proper procedures or control of the release of these virtual machines. [Source](https://www.techrepublic.com/blog/virtualization-coach/what-is-your-best-definition-of-vm-sprawl)

### golden images

Once software has been compiled and tested and re-tested, the perfect build is declared gold. No further changes are allowed, and all distributable copies are generated from this master image (this used to actually mean something, back when software was distributed on CDs or DVDs). [Source](https://opensource.com/article/19/7/what-golden-image)

### cascading failure
A cascading failure is a process in a system of interconnected parts in which the failure of one or few parts can trigger the failure of other parts and so on. Such a failure may happen in many types of systems, including power transmission, computer networking, finance, transportation systems, organisms, the human body, and ecosystems.

Cascading failures may occur when one part of the system fails. When this happens, other parts must then compensate for the failed component. This in turn overloads these nodes, causing them to fail as well, prompting additional nodes to fail one after another. [Wiki](https://en.wikipedia.org/wiki/Cascading_failure)

### race condition
A race condition or race hazard is the condition of an electronics, software, or other system where the system's substantive behavior is dependent on the sequence or timing of other uncontrollable events. It becomes a bug when one or more of the possible behaviors is undesirable. [Wiki](https://en.wikipedia.org/wiki/Race_condition)

### Load shedding
Load shedding is a technique that allows your system to serve nominal capacity, regardless of how much traffic is being sent to it, in order to maintain availability. To do this, you'll need to throw away some requests and make clients retry. [Source](https://cloud.google.com/blog/products/gcp/using-load-shedding-to-survive-a-success-disaster-cre-life-lessons)

### Graceful degradation
Graceful degradation takes the concept of load shedding one step further by reducing the amount of work that needs to be performed. In some applications, it’s possible to significantly decrease the amount of work or time needed by decreasing the quality of responses. For instance, a search application might only search a subset of data stored in an in-memory cache rather than the full on-disk database or use a less-accurate (but faster) ranking algorithm when overloaded. [Source](https://sre.google/sre-book/addressing-cascading-failures/#xref_cascading-failure_load-shed-graceful-degredation)

### Exponential Backoff
In a variety of computer networks, binary exponential backoff or truncated binary exponential backoff refers to an algorithm used to space out repeated retransmissions of the same block of data, often to avoid network congestion.[Wiki](https://en.wikipedia.org/wiki/Exponential_backoff)

### Query of Death
an RPC whose contents trigger a failure in the process [Source](https://sre.google/sre-book/addressing-cascading-failures/#process-death)

### CAP Theorem
The logic is intuitive: if two nodes can’t communicate (because the network is parti‐
tioned), then the system as a whole can either stop serving some or all requests at
some or all nodes (thus reducing availability), or it can serve requests as usual, which
results in inconsistent views of the data at each node.

### TCP slow start
TCP slow start is an algorithm which balances the speed of a network connection. Slow start gradually increases the amount of data transmitted until it finds the network’s maximum carrying capacity. [Source](https://blog.stackpath.com/tcp-slow-start)

### round trip time RTT

Round Trip Time (RTT) is the length time it takes for a data packet to be sent to a destination plus the time it takes for an acknowledgment of that packet to be received back at the origin. The RTT between a network and server can be determined by using the ping command. [Source](https://developer.mozilla.org/en-US/docs/Glossary/Round_Trip_Time_(RTT)#:~:text=Round%20Trip%20Time%20(RTT)%20is,by%20using%20the%20ping%20command.)

### replicated state machine
In computer science, state machine replication or state machine approach is a general method for implementing a fault-tolerant service by replicating servers and coordinating client interactions with server replicas. The approach also provides a framework for understanding and designing replication management protocols [Wiki](https://en.wikipedia.org/wiki/State_machine_replication)

### Byzantine fault tolerance
The objective of Byzantine fault tolerance is to be able to defend against failures of system components with or without symptoms that prevent other components of the system from reaching an agreement among themselves, where such an agreement is needed for the correct operation of the system.

In a Byzantine fault, a component such as a server can inconsistently appear both failed and functioning to failure-detection systems, presenting different symptoms to different observers. It is difficult for the other components to declare it failed and shut it out of the network, because they need to first reach a consensus regarding which component has failed in the first place.

Byzantine fault tolerance (BFT) is the dependability of a fault-tolerant computer system to such conditions. [Wiki](https://en.wikipedia.org/wiki/Byzantine_fault)

### consensus algorithm
A fundamental problem in distributed computing and multi-agent systems is to achieve overall system reliability in the presence of a number of faulty processes. This often requires coordinating processes to reach consensus, or agree on some data value that is needed during computation. Example applications of consensus include agreeing on what transactions to commit to a database in which order, state machine replication, and atomic broadcasts.


### edge cases
An edge case is a problem or situation that occurs only at an extreme (maximum or minimum) operating parameter. For example, a stereo speaker might noticeably distort audio when played at maximum volume, even in the absence of any other extreme setting or condition. [Wiki](https://en.wikipedia.org/wiki/Edge_case)

### secret sprawl
passwords hardcoded in apps, stored in multiple configs all over the infrastructure

### noizy neighbour
when tenant utilize all resources in multitenant env

### Merging strategies: Git Flow
The Gitflow Workflow defines a strict branching model designed around the project release. This provides a robust framework for managing larger projects. [Source](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow#:~:text=Gitflow%20Workflow%20is%20a%20Git,development%20and%20implementing%20DevOps%20practices.&text=The%20Gitflow%20Workflow%20defines%20a,framework%20for%20managing%20larger%20projects.)

### Merging strategies: Trunk Based Development
A source-control branching model, where developers collaborate on code in a single branch called ‘trunk’ *, resist any pressure to create other long-lived development branches by employing documented techniques. They therefore avoid merge hell, do not break the build, and live happily ever after. [Source](https://trunkbaseddevelopment.com/)

### Configuration drift
Configuration drift is a data center environment term. At a high level, configuration drift happens when production or primary hardware and software infrastructure configurations “drift” or become different in some way from a recovery or secondary configuration or visa versa. [Source](https://www.continuitysoftware.com/blog/it-resilience/what-is-configuration-drift)


### Progressive delivery
It is essentially a modified version of continuous delivery – in fact, before the term “progressive delivery”, many people called it “continuous delivery ++” – with two core differences. First: progressive delivery teams use feature flags to increase speed and decrease deployment risk. Second: they implement a gradual process for both rollout and ownership.[Source](https://www.split.io/glossary/progressive-delivery/)
