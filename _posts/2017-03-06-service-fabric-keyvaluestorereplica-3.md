---
layout: post
title: "Service Fabric – KeyValueStoreReplica, Prefix query and actors"
date: 2017-03-06 09:55
author: scooletz
permalink: /2017/03/06/service-fabric-keyvaluestorereplica-3/
nocomments: true
image: /img/2017/03/scooletz_image.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

[The last post](http://blog.scooletz.com/2017/03/02/service-fabric-keyvaluestorereplica-2) presented a deep dive into capabilities of internal Service Fabric storage. We saw, that in spite of being "just a key value store", *KeyValueStoreReplica* enables transactions, optimistic concurrency and an interesting approach for handling queries. It's time to go back to Service Fabric actor model and understand how Actors have their state persisted.

### Actor state manager revisited

To access its state Service Fabric actors use [IActorStateManager](https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicefabric.actors.runtime.iactorstatemanager), which enables to setting and getting states by their names. The manager provides one more method, that is called by the actor class when handling a call is ended this method is:

```csharp
Task SaveStateAsync(CancellationToken token)
```

This is the place where the actor state is persisted. Let's take a look at the implementation details behind the default actor state manager.

For persisted actors, the call eventually lands in the *KvsActorStateProvider* class which is nothing more than just a wrapper around *KeyValueStoreReplica*. During *SaveStateAsync* call:

1. *KeyValueStoreReplica* transaction is opened

1. All the states that were changed/added/removed have appropriate methods invoked (Update, Add, Remove)
1. Transaction is committed.

It's all good, but how to ensure that one can easily access all the states of a specific actor? How to make it easy and accessible with one call?

### The key is... the key

Because KeyValueStoreReplica provides no indexes and accepts just a string as the key, you could think that the key formatting is crucial to ensure the ability to query I mentioned before. That's true.

To make it happen the following algorithm is used to encode the key

```csharp
var key = $"Actor_{actorId}_{stateName}"
```
As you can see every single actor state starts with the "Actor_" prefix. Then, the *actorId* is used, then the state name is appended. This means that every state of the actor has the same name, which means that we could use [Prefix query](https://blog.scooletz.com/2017/03/02/service-fabric-keyvaluestorereplica-2) with the following prefix to make it happen:

```csharp
var prefix = $"Actor_{actorId}"
```

### Summary

After reading this post you should understand how actors use the underlying Service Fabric store and how they ensure that all the values of a specific actor instance are collocated to make them easy to access together.
