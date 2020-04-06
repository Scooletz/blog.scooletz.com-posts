---
layout: post
title: "Why persistent memory will change your world"
date: 2016-12-07 09:55
author: scooletz
permalink: /2016/12/07/why-persistent-memory-will-change-your-world/
image: /img/2016/12/stocksnap_1wkw24wdx7.jpg
categories: ["Design", "Hardware", "Uncategorized"]
tags: ["design", "persistent memory"]
imported: true
---

### TL;DR

If you haven't heard, the non volatile RAM memory is coming to town and for sure will change the persistence patterns of databases, queues, loggers. Want to know more about this new wave of hardware? Read along.

### API

The first and the most important aspect is that persistent memory on Windows, reuses already existing APIs. First, if you want to use the drive just as a block storage, you can do it. You'll be able to create files, write them etc. There's another way of using it, which is much faster, called DAX.

The direct access enables to use the non volatile memory directly. What do I mean by *directly*? I mean accessing the memory with a raw pointer. How do you obtain a pointer? The old fashioned memory mapped file API is used. First, create a file, then map it and here it is! No *FlushFileBuffers*, no *fsync*. Just a raw pointer to the memory. Can you imagine writing to a mapped file and just having it persisted?

### Speed

The non volatile memory is really fast. You can write 4GB per second. Yes, it's 4GB per second of a persistent memory. The latency is extremely low. It's so low, that using any form of asynchronous programming (raw completion ports, async-await) brings more havoc than just waiting for having this memory written. Yes, this means that your methods hitting files mem mapped with DAX will not need async signatures. Of course you'll be able to preserve them just for the compatibility.

### Ordering matters

It looks like tech heaven. No more data loss during power outages, right? It's not entirely true. The persistent memory acts as a memory. There is an order of execution in which data are transported there. Imagine now writing the following string:

*BLAH*

If power went down after copying first three letters you'd be left with

*BLA*

Which although shows the same attitude, is not what we wish for when thinking about persistence. This example shows, that good old fashioned IO access patterns will still be important, like *write-ahead logging* or *copy-on-write*. But let me remind you again, they will be free with no synchronization, no flushing required.

### Adopters

The persistent memory will change the world. SQL Server 2016 has already adopted it, as you can see [here](https://channel9.msdn.com/Shows/Data-Exposed/SQL-Server-2016-and-Windows-Server-2016-SCM--FAST). Some databases are already there, like [LMDB](https://symas.com/products/lightning-memory-mapped-database/) using memory mapped files (same API for the win!) with an ability to run as non-durable. Guess what. Now it's durable. More databases will follow.

### Summary

The persistent memory is here. You won't probably rewrite or rethink your application as majority of apps do not deal with IO directly, but just by applying it to your database or other IO bound systems will be a real game changer.
