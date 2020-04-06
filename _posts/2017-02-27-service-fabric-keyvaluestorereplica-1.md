---
layout: post
title: "Service Fabric - KeyValueStoreReplica (1)"
date: 2017-02-27 09:55
author: scooletz
permalink: /2017/02/27/service-fabric-keyvaluestorereplica-1/
nocomments: true
image: /img/2017/02/scooletz_image.png
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

In the previous post I introduced [Sewing Machine](http://blog.scooletz.com/2017/02/23/sewing-machine-for-service-fabric), a helper library for building more on top of strong Service Fabric foundations. Before building a thing, one needs to [know the foundations](http://blog.scooletz.com/2017/02/13/my-learning-pattern/) though. That's where we start our Service Fabric journey.

### Actors

One of the paradigms that are supported by Service Fabric is a Virtual Actor pattern. Actors are simple and small single threaded executions, that have their own state and a lifecycle. The *Virtual Actor pattern* ensures that you never need to instantiate any actor. Once you access an actor via its id, it will be created and from now on hosted somewhere in the cluster. Where? That depends on the balancer. All you need to know is an actor type and its identifier.

More about actors you can read in a good introduction provided in [here](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-actors-introduction)

### Actors state

Service Fabric actors can have multiple states. They are accessible by the [StateManager property](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-actors-state-management) almost like a regular dictionary. If the actor is marked as *Persisted,* you have a guarantee that its state will be stored between calls, ensuring that in case of a machine failure, it can be restored on another one without data loss.

        [StatePersistence(StatePersistence.Persisted)]
class CountingActor : Actor, ICountingActor
{
  public Task SetCountAsync(int value)
  {
    return this.StateManager
      .SetStateAsync("Counter", value);
  }
}

Additionally, multiple states of the same actor can be changed simultaneously and will be stored in a transaction. Yes, you can't run into a situation with a partially stored state of a persisted actor.

Why would one store different states then? Think of it as a simple partitioning your actors' data. If you don't need the whole state every time, maybe it's good to split them according to their use cases.

The *StateManager* behind the curtain implements a simple Unit of Work. If you access, set or delete the state under the same key within one call, the StateManager will track data and flush it properly in a transaction at the end of the call.

### But transaction means db

If we say *transaction*, it means that there must be a database behind it. This is the case of *Persisted* actors as well. Behind the interface of [IActorStateProvider](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.actors.runtime.iactorstateprovider) providing the API for Actor state management, its implementation calls a very particular class providing access to *a transactional, replicated, associative data storage component to service writers - ready for integration into any Service Fabric service*. This is the database responsible for storage. It's called [KeyValueStoreReplica](https://docs.microsoft.com/en-us/dotnet/api/system.fabric.keyvaluestorereplica).

### Summary

In this first blog post we followed from the high level API of actors' framework, through the state management to the underlying database. In the following posts we'll dive deeper into capabilities provided by this storage.
