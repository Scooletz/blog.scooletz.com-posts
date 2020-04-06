---
layout: post
title: "Service Fabric – KeyValueStoreReplica, ReplicaRole"
date: 2017-03-09 21:55
author: scooletz
permalink: /2017/03/09/service-fabric-keyvaluestorereplica-replicarole/
nocomments: true
image: /img/2017/03/scooletz_image2.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

After taking a look at how actors' state is persisted with KeyValueStoreReplica to follo [the prefix query](http://blog.scooletz.com/2017/03/06/service-fabric-keyvaluestorereplica-3) guideline, it's time to see how this state is replicated.

### ReplicaRole

When defining replication for a partition, one defines on how many nodes the data will reside. Every copy of a partition's data is called Replica. It's important to know, that for a given partition only one replica at a time is active. This kind of replica is called Primary. Let's take a look at the [ReplicaRole](https://msdn.microsoft.com/en-us/library/azure/dn707635.aspx) values and decipher their meanings:

1. Primary - the currently active replica. All operations are handled by the primary, ensuring that any write will be replicated and acknowledged by a quorum of *ActiveSecondary* replicas. As in The Highlander, there can be only one *Primary* replica at the same time.

1. IdleSecondary - a replica that accepts and applies a state send by the *Primary* to catch up with all the changes and eventually become *ActiveSecondary* as soon as it catch up.

1. ActiveSecondary - a replica that is a part of the write quorum. It stores updates from the *Primary* and acknowledge them to enable *Primary* to successfully end a write operation.

### Active, passive and not-that-passive

As you can see above, there's only one replica at any time that is truly active, it's Primary. What happens with the secondaries? Can they do anything meaningful or maybe they're just there for copying state?

First and foremost, secondary replicas receive notifications about the state being replicated. This means, that if you derive from *KeyValueStoreReplica* class, you can be notified about the copied key-value pairs. That's how you can react to these changes. But how would this be helpful?

You could index the data somehow, you could notify other services, endpoints calling their methods or sending a request (in a safe manner, not failing on the notification) and much more. For instance, Actors' Runtime uses it to capture the last timestamp for a component called VolatileLogicalTimeManager.

### Summary

The role of a replica can be easily summarized as: primary - the current active replica accepting reads&writes, secondary - replicas just copying the state.
