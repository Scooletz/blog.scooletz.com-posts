---
layout: post
title: "Reconcilation between systems"
date: 2014-06-06 09:00
author: scooletz
permalink: /2014/06/06/reconcilation-between-systems/
nocomments: true
categories: ["Architecture", "Design"]
tags: ["Cassandra", "migration", "reconcilation", "system migration"]
imported: true
---

From time to time a system is replaced with another system being capable of doing more, or doing the thing better. It's quite to common to ask whether no data is lost or does the system preserve needed behaviors of the old one. Sometimes it's human-application comparison, when a procedure followed by people is replaced with an application, sometimes it's a question of an old system vs a new system

### Cassandra integrity check

Let's presume one migrates some old-fashioned SQL system to a Cassandra based solution given following:

1. total payload of daily data is quite high
1. data are written to the Cassandra cluster (more than one node) with ConsistencyLevel Local Quorum
1. there should be a possibility to check whether all the data stored in the previous systems are written to the new one

After a bit of consideration one can propose that as the data are written with *LocalQuorum*, they should be queried with the same level and match in the old solution. This would ensure that data which has been written are being read (famous *[R + W > N](http://wiki.apache.org/cassandra/ArchitectureOverview)*). This could cost a lot as querying hits [N+1]/2 nodes of your cluster, streaming a daily payload through network twice: once to the coordinator, second - to the client. Can we do this better?

### Possibly faster integrity check

How about using Consistency Level of **One**? How can this be done to ensure that the given node consists of all the needed data? By running repair in your local data center on each node, one can ensure that each node consist of all the data it's responsible for. Then, querying with One is ok. What's important about [nodetool repair](http://www.datastax.com/documentation/cassandra/2.0/cassandra/tools/toolsRepair.html) is that it does not stream data if it's not needed. The information sent to match if the given node contains all the data is a [Merkle tree](http://en.wikipedia.org/wiki/Merkle_tree), a tree made by hash of hashes of hashes of... Sending this structure is cheap and doesn't your network so much.
If you consider (know that) running repairs daily is a heavy task for your cluster, you'll be happy to read about Cassandra 2.1 repair improvements, including [incremental repairs](http://www.datastax.com/dev/blog/more-efficient-repairs).

So stop complaining about your good old fashioned RMDB and get yourself a new shiny cluster of Cassandra nodes :)
