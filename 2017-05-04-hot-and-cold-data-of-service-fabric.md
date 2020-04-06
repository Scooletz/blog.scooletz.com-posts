---
layout: post
title: "Hot or not? Data inside of Service Fabric"
date: 2017-05-04 08:55
author: scooletz
permalink: /2017/05/04/hot-and-cold-data-of-service-fabric/
nocomments: true
image: /img/2017/04/stocksnap_eizqlk79fe.jpg
categories: ["Azure", "Service Fabric"]
tags: ["azure"]
imported: true
---

### TL;DR

When calculating the space needed for your Service Fabric cluster, especially in Azure, one can hit machine limits. After all, a D2 instance has only 100 GiB of local disk and this is the disk used by Service Fabric to store data onto. 100 GiB might be not that small, but if you use your cluster for more than one application, you can hit the wall.

### Why local?

There's a reason behind using local, ephemeral disk for Service Fabric storage. The reason is locality. As Service Fabric replicates the data, you don't need to store them in a highly available storage as the cluster provides one on its own. Storing data in multiple copies by using Azure Storage Services is not needed. Additionally, using local, SSD drives is much faster. It's a truly local disk after all.

### Saturation

Service Fabric is designed to run many applications with many partitions. After all, you want to keep your cluster saturated (almost) as using a few VMs just to run an app that is needed once in a month would be useless. Again, if you run many applications, you need to think about capacity. Yes, you might be running stateless services which don't require one, but not using stateful services would be a waste. They provide an efficient, transactional, replicated database built in inside of them. So what about the data? What if you saturate the cluster not in terms of CPU but the storage.

### Hot or not

One of the approaches you could use is the separation between hot and cold data. For instance, users haven't logged in for one month could have their data considered as cold. These data could be offloaded from the cluster to Azure Storage Services, leaving more space for one that are needed. When writing applications that use an append only model (for instance ones based on event sourcing) you could think about offloading events older than X days, at the same time ensuring that they can be accessed. Yes, the access will be slower, but it's unlikely that you'll need them on regular basis.

### Summary

When designing your Service Fabric apps and planning your cluster capacity think through the hot/cold approach as well. This, could lower your requirements for the storage space and enable you to use the same cluster for more application, which effectively is what Service Fabric is for.
