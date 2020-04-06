---
layout: post
title: "Service Fabric – KeyValueStoreReplica (2)"
date: 2017-03-02 09:55
author: scooletz
permalink: /2017/03/02/service-fabric-keyvaluestorereplica-2/
nocomments: true
image: /img/2017/02/scooletz_image.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

In [the previous post](http://blog.scooletz.com/2017/02/27/service-fabric-keyvaluestorereplica-1/) we started the top bottom journey into the Service Fabric actor model. The very last step was encountering *KeyValueStoreReplica* **that provides persistence capabilities for persisted actors. Today, we'll review the capabilities of this distributed, associative data storage.

### Transactional

The first and quite interesting attribute of *KeyValueStoreReplica* is its transactionality. You can open a transaction by calling a regular *CreateTransaction* method and later on commit it with *CommitAsync* like in the following example.

```csharp
using (var tx = replica.CreateTransaction ())
{
  // do something
  var seqNumber = await tx.CommitAsync()
                          .ConfigureAwait(false);
}

```

This is the way actors ensure that all the states are stored atomically. No partially stored state, it's all or nothing.

As you probably noticed, there's a *sequence number* that is returned when a transaction is committed. We'll get back to it shortly.

### Key Value store

*KeyValueStoreReplica* is a key value store. Although it's transactional as we saw above, it can store only binary payloads accessed by string keys. You can easily:
* Add / TryAdd
* Update / TryUpdate
* Remove / TryRemove
* Get / TryGet

a value by passing the ongoing transaction (you can't do anything without an active transaction), the key and the value when needed. See the following example:
```csharp
using (var tx = r.CreateTransaction ())
{
  r.Add (tx, "key1", new byte[] {1,2});
  r.Update (tx, "key2", new byte[] {3,4});
  var seqNumber = await tx.CommitAsync()
                          .ConfigureAwait(false);
}
```
There's an additional parameter that you can pass to all the methods changing data. It's a *sequence number*.

### Sequence number

The sequence number is simply an optimistic concurrency marker. Once you retrieve it either by getting it with data when *Getting* value or obtain it from a committed transaction, you can use it to *Update* or *Delete* values conditionally. If between your first call that resulted in obtaining the sequence number and the second used for *Update* or *Delete* something changed any value accessed in the second transaction it will fail.

This pattern may lower the need of rereading data every time and simply use the marker returned from the last committed transaction.

### Prefix query

Although *KeyValueStoreReplica* is a key-value store it provides one additional way of querying data instead of getting the values one by one. This feature is called a prefix query. Consider the following example where two values are added with keys that have a common prefix. They can be retrieved in one call and returned as an enumerator

```csharp
using (var tx = r.CreateTransaction ())
{
   r.Add (tx, "key1", new byte[] {1,2});
   r.Add (tx, "key2", new byte[] {3,4});
   var enumerator = r.Enumerate(tx, "key");
   // enumerator has: "key1", "key2"
}
```

### Summary

In this blog post we saw that the underlying storage for stateful part of the Service Fabric cluster is much more than a dummy key value store. It is a transactional db, it enables optimistic concurrency and enables a prefix query that with a proper key design can be leveraged to do a lot. But this is a topic for another blog post.
