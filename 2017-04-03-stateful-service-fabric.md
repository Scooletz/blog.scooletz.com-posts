---
layout: post
title: "Stateful Service Fabric"
date: 2017-04-03 08:55
author: scooletz
permalink: /2017/04/03/stateful-service-fabric/
nocomments: true
image: /img/2017/04/scooletz_image.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

This is a very next entry in a series covering my journey based on Service Fabric and my Open Source project called [SewingMachine](https://github.com/Scooletz/SewingMachine). After hitting [the internal wall of Service Fabric](http://blog.scooletz.com/2017/03/13/hitting-internal-wall-in-service-fabric/) I pivoted the approach of as I knew that I can't change the Actors' part as I wanted (too much internals) and need to work on a different level.

### Underneath it's all stateful

It doesn't matter if you use [StatefulService](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.services.runtime.statefulservice) or the actor's model of Service Fabric. In the first case you explicitly work with a state using reliable collections: a distributed, transactional dictionary and a queue. You can access them using *StateManager* of your service derived from the mentioned base class.

In case of actors, you just register an actor in the Actors' Runtime. Underneath, this results in creating a stateful [ActorService](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.actors.runtime.actorservice). Both, *StatefulService* and *ActorService* derive from *StatefulServiceBase*. The way they provide and operate on the state is quite different.

### Reliable collections

Reliable collections provide a dictionary and a queue. Both, are transactional so you can wrap any operations with a regular transaction. They provide different semantics, where the dictionary is a regular dictionary, a lookup if you prefer or a hashmap. The queue is a regular FIFO structure. The transactions have a bit different semantics from regular db transactions, but for now, you may treat them as regular ones.

If the stateful services use reliable collections, is it the same for actors? It looks like it's not.

### KeyValueStoreReplica

The storage for actors is provided by KeyValueStoreReplica already covered in [here](http://blog.scooletz.com/2017/02/27/service-fabric-keyvaluestorereplica-1/) and [here](http://blog.scooletz.com/2017/03/02/service-fabric-keyvaluestorereplica-2/). If you asked *is it that much different from the reliable collections*, I'd say yes. It has different semantics and provides a simple replicable key-value store. If its is so different, could we build something else than an actor system on top of it?

The answer will be revealed in the very next post.

###
