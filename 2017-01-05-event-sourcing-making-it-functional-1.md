---
layout: post
title: "Event sourcing: making it functional (1)"
date: 2017-01-05 09:55
author: scooletz
permalink: /2017/01/05/event-sourcing-making-it-functional-1/
image: /img/2016/12/imag0271.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "event sourcing"]
imported: true
---

### TL;DR

This article starts a series of entries that will guide you through my experiences with making event sourcing functional. There are a few existing entries about a functional approach to event sourcing, but I want to share my path and a story behind migrating from one approach to another. I'll start with some fundamentals. This will show from where I started, as well as it may help you to learn some basics of DDD and event sourcing if you're not into the topic.

### All entries in this series:

* 1 - back to foundations
* [2 - aggregate sketch](http://blog.scooletz.com/2017/01/09/event-sourcing-making-it-functional-2/)
* [3 - first implementation](http://blog.scooletz.com/2017/01/12/event-sourcing-making-it-functional-3/)
* [4 - finding events](http://blog.scooletz.com/2017/01/16/event-sourcing-making-it-functional-4/)
* [5 - applying events](http://blog.scooletz.com/2017/01/19/event-sourcing-making-it-functional-5/)
* [6 - recording aggregate](http://blog.scooletz.com/2017/01/23/event-sourcing-making-it-functional-6/)
* [7 - the fully functional end](http://blog.scooletz.com/2017/01/26/event-sourcing-making-it-functional-7/)

### The Blue Book![51szw87slrl-_sx258_bo1204203200_](/img/2016/12/51szw87slrl-_sx258_bo1204203200_.jpg)

There's a book that is a must read, **Domain-Driven Design by Eric Evans**. It's quite old and a lot of has changed in our industry. Still, the basic need of having developers understanding the domain they work on, the strategies and tactics they could use to embrace it, they are the same. Eric covers a lot of topics in there, one of them which is a really low level concept (and not the most important one) is the aggregate.

### The aggregate

Let me quote Eric's descriptions of the aggregate first:

*An AGGREGATE is a cluster of associated objects that we treat as a unit for the purpose of data changes. Each AGGREGATE has a root and a boundary. The boundary defines what is inside the AGGREGATE. The root is a single, specific ENTITY contained in the AGGREGATE. The root is the only member of the AGGREGATE that outside objects are allowed to hold references to, although objects within the boundary may hold references to each other. ENTITIES other than the root have local identity, but that identity needs to be distinguishable only within the AGGREGATE, because no outside object can ever see it out of the context of the root ENTITY.*

The vital part of this description is the boundary. That's what aggregate is for - to distill boundaries existing in a domain you work on. So once you set a boundary of data changed together, you ensure that they stay together. Additionally, you operate on this data only via the **Aggregate Root**, which might be treated as the public API of an aggregate, something that others interacts with.

### The aggregate boundaries

If an aggregate is defined by its boundaries, one could design a system, that would be just one aggregate. This would enable to access all the data and still have a defined boundary, right? Wrong. What you want to do is to define aggregates following these two simple rules:

1. aggregates are as big as there have to
1. aggregates are as small as possible

### Aggregates are as big as there have to

The very first rule says that the size of an aggregate is determined by a domain and a model. A domain can require some consistency across different entities, making an aggregate bigger. The same with model. Frequently it's up to the modeller (a model's creator) to create bigger or smaller aggregate, but not bigger than needed.

### Aggregates are as small as possible

Distilling just one aggregate would mean, that a system can execute one action at a time (when optimistic concurrency applied). This isn't enough for the majority of applications. Making aggregates small improves your system's scalability and performance, not to mention your ability to design and model in small chunks.

### Summary

This ends the first entry of this series. We recalled the aggregate's definition and discussed briefly two rules of modelling an aggregate. Still, there's more to come on our way of making the event sourcing functional. First, we need to visit a non-functional approach.
