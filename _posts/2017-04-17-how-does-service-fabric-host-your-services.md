---
layout: post
title: "How does Service Fabric host your services?"
date: 2017-04-17 08:55
author: scooletz
permalink: /2017/04/17/how-does-service-fabric-host-your-services/
nocomments: true
image: /img/2017/04/stocksnap_5lgt5k24k6.jpg
categories: ["Azure", "Service Fabric"]
tags: ["azure"]
imported: true
---

### TL;DR

Service Fabric provides an amazing fully automated hosting for any number of services with any number of instances each (up to the physical limits of your cluster). But how are these hosted? What if you have more partitions than nodes?

### Structure recap

When building an app that is meant to be hosted in Service Fabric you build an... app. This application might consist of multiple stateful and stateless services. The application is packaged to a... package that, when uploaded to the cluster as an image, provides an application type. From this, you can instantiate multiple application instances. Let me give you an example.

Application "Bank" consists of two services:

1. "Users"
1. "Accounts"

When build with version "1.0.0" and packaged, it can be uploaded to the SF cluster and is registered as "Bank 1.0.0". From now on you can instantiate as many banks as you want within your cluster. Each will be composed of two sets of services: "Users" and "Accounts".

### Services, stateful services

When defining stateful services, these that have a built in kind-of database (reliable collections, or SewingSession provided by my project [SewingMachine](https://github.com/Scooletz/SewingMachine)) you need to define how many partitions they will have. You can think of partitions as separate databases. Additionally, you define the number of replicas every partition will have. That's done to ensure high availability. Let me give you an example.

1. "Users" have the number of partitions set to 100 and every partition is replicated to 5 replicas (let's say P=100, R=5)
1. "Accounts" are configured with P=1000, R=7

Imagine that it's hosted on a cluster that has only 100 nodes. This will mean, that on every node (on average) system will place 5 replicas of "Users" and 70 replicas of "Accounts". It's a valid scenario. Once some nodes are added to the cluster, replicas will be automatically moved to new nodes lowering the saturation of previously existing.

What if a node hosts more than one replica of one service, how are they hosted? Moreover, how do the communicate, as there's only one port assigned to do it?

### Cohosting to the rescue

Currently, all the replicas are hosted within the same process. Yes, although 5 "Users" instances will be created, they will all be sitting in the same AppDomain of the same process. The same goes for 70 "Accounts". You can check it on your own by obtaining the current process ID (PID) and *AppDomain.Current* and compare. This reduces the overhead of hosting as all assemblies and static resources (assemblies loaded, static fields, types) are shared across replicas.

### One port to rule them all

By default, when using native Service Fabric communication listener, only one port is used by an endpoint. How is possible that the infrastructure knows how to route messages to the right partition and replica? Under the hood, when opening a communication listener, replica registers the identifier of the partition it belongs to and its replica number. That's how, when a message arrives, Service Fabric infrastructure is capable of sending the message to the right communication listener, and therefore, to the right service instance.

### Summary

Now you know, that all replicas of partitions of the same service on one node are cohosted in the same process and that Service Fabric infrastructure dispatches messages accordingly to the registered partition/replica pair.
