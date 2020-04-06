---
layout: post
title: "Hitting internal wall in Service Fabric"
date: 2017-03-13 09:55
author: scooletz
permalink: /2017/03/13/hitting-internal-wall-in-service-fabric/
image: /img/2017/03/stocksnap_637f0e57b1.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

In this post I share my experience in trying extending Service Fabric for [Sewing Machine](https://github.com/Scooletz/SewingMachine) purposes.

### Sewing Machine aim

The aim of Sewing Machine is to extend the Service Fabric actor model to use better, faster, less-allocating foundations. The only part I was working on so far was persisted actors. This is the one stored in the KeyValueReplica, I've been writing about for last few weeks.

### Not-so public seam

I started my work by discovering how the persisted actors are implemented. The class responsible for it is named *KvsActorStateProvider*. It uses following components:

* KeyValueStoreWrapper - private
* VolatileLogicalTimeManager.ISnapshotHandler - internal
* VolatileLogicalTimeManager - internal
* IActorStateProviderInternal - internal
* ActorStateProviderHelper - internal, responsible for shared logic among providers
* IActorStateProvider - public interface to implement

As you can see, the only part that is given is a seam of the state provider. Every single helper that one could use to implement their own, is internal. Additionally, the provider interface is filled with other interfaces that one needs to implement. I know, sharing data structures isn't the best option, but as the ServiceFabric share them internally, why wouldn't you give it to the user.

All of the above means that extending actors' runtime is hard if not impossible. It provides no real extension points and has its public seam not prepared for it. What does it mean for SewingMachine?

### Can't win, change battle

Rewriting the runtime would be time-consuming. I can't spend half a year on writing it and don't want to. At least, not now. I've implemented a faster unsafe wrap around KeyValueReplicaStore that still can be useful. To make an impact with SewingMachine, I'll introduce the event driven actor first, even when clean up will be run by not efficient regular Actor disposal. Using a custom serializer and adhering to the currently used prefixes, later on can be changed to use a custom runtime. The only problem would be reminders, but this can be handled as well by a better versioning of them.

### Summary

SewingMachine was meant to extend the actors' runtime. Seeing difficulties like the ones described above, I could either kill it or repurpose it to provide a real value first, leaving performance for later. That's how we'll do it.
