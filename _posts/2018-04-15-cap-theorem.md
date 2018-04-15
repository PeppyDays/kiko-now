---
layout: post
title: CAP Theorem
tags: [cap, consistency, availability, partition, tolerance]
---

CAP theorem states that it is impossible to simutaneously provide more than two out of three things **when network failures** - consistency, availability and partition tolerance.

Refer to [CAP_Theorem Wiki](https://en.wikipedia.org/wiki/CAP_theorem).

## Trade-off items

Consistency, availability and partition tolerance

### Consistency

Every read receives the most recent write or an error.

### Availability

Every request receives a response, not error.

### Partition Tolerance

The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes.

## Explanation

Assume that this system is a host distributed system, and it needs network connection each other. No distributed system is safe from network failures, thus network partitioning generally has to be tolerated.

In the presence of a partition, one is then left with two options: consistency or availability.

When choosing consistency, the system will return an error or a time-out if particular information cannot be guaranteed to be up to date due to network partitioning.

When choosing availability, the system will always process the query and try to return the most recent available version of the information, even if it cannot guarantee it is up to date due to network partitioning.

In the absence of network failure – that is, when the distributed system is running normally – both availability and consistency can be satisfied.

CAP is frequently misunderstood as if one has to choose to abandon one of the three guarantees at all times. In fact, the choice is really between consistency and availability only when a network partition or failure happens; at all other times, no trade-off has to be made.