---
layout: post
title: "Multidatacenter Cassandra cluster with slow cross DC connection"
date: 2014-04-07 11:00
author: scooletz
permalink: /2014/04/07/multidatacenter-cassandra-cluster-with-occasional-network-partitions/
nocomments: true
categories: []
tags: ["Cassandra", "network partition", "split brain", "splitbrain"]
imported: true
---

I'd like to discuss a particular failure scenario for a multi datacenter Cassandra cluster.
The setup to reproduce is following:

* Two Cassandra data centers
* *   DC1: n nodes

    *   DC2: m nodes
* TestKeyspace
* NetworkTopologyStrategy with replication factors:
* *   DC1: n (each key on each node)

    *   DC2: m (each key on each node)
* Tables in TestKeyspace are created with default settings
* hinted hand-off enabled
* read repair enabled

The writes and reads goes to the DC1. What can go wrong when whole DC2 goes down (or you get a network split)?

It occurs that read_repair is defined not by one but two probabilities:

* [read repair chance](http://www.datastax.com/docs/1.1/configuration/storage_configuration#read-repair-chance) with default value of 0.1
* [dclocal read repair chance](http://www.datastax.com/docs/1.1/configuration/storage_configuration#dclocal-read-repair-chancehttp://) with default value of 0.0

What's the difference between them? The first one shows probability of read repair across whole cluster, the second - rr across the same DC. If you have an occasionally failing connection, or a slow one using the first can bring you some troubles. If you plan for multi DC cluster and you can live with periodical runs nodetool repair instead of failing some of your LOCAL_QUORUM reads from time to time, switch to dc read repair and disable the global one.

For curious readers the class responsible for performing reads with read-repairs as well is [AbstractReadExecutor](https://github.com/apache/cassandra/blob/72577cee6e0f7b84de468ea75e609db30fd8a9d2/src/java/org/apache/cassandra/service/AbstractReadExecutor.java)
