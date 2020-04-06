---
layout: post
title: "Better Stateful Service Fabric"
date: 2017-04-06 08:55
author: scooletz
permalink: /2017/04/06/better-stateful-service-fabric/
nocomments: true
image: /img/2017/04/scooletz_image_better.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

This is a follow up entry to [Stateful Service Fabric](http://blog.scooletz.com/2017/04/03/stateful-service-fabric) that introduced all the kinds of statefulness you can out of the box from the Fabric SDK. It's time to move on with providing a better stateful service. First, we need to define what better means.

### KeyValueStoreReplica implementation details

The underlying actors' persistence is based on *KeyValueStoreReplica*. This class provides a high level API for low level interop interfaces provided by Service Fabric. Yep, you heard it. The runtime of the Fabric is unmanaged and is exposed via COM+ interfaces. Then they are wrapped in a nice .NET classes just to be delivered to you. The question is how nice are these classes? Let's take a look at the decompiled part of a method.

```csharp

private void UpdateHelper(TransactionBase transactionBase, string key, byte[] value, long checkSequenceNumber)
{
using (PinCollection pin = new PinCollection())
{
IntPtr key1 = pin.AddBlittable((object) key);
Tuple<uint, IntPtr> nativeBytes = NativeTypes.ToNativeBytes(pin, value);
this.nativeStore.Update(transactionBase.NativeTransactionBase, key1, (int) nativeBytes.Item1, nativeBytes.Item2, checkSequenceNumber);
}
}

```



### pinning + Interop + pointers = fun

As you can see above, when you pass a *string* key and a *byte[]* value a lot must be done to execute the update:

1. *value* needs to be allocated every time

    as there's no interface accepting *Stream* or *ArraySegment<byte>* that would allow you to reuse bytes used for allocations, you always need to *ToArray* the payload

1. *key* and *value* are pinned for the time of executing update.

    Pinning, is nothing more or less than prohibiting object from being moved by GC. The same effect that you get when using *fixed* keyword or using [GCHandle.Alloc](https://msdn.microsoft.com/en-us/library/a95009h1(v=vs.110).aspx). When handles not that many requests it's ok to do it. When GC kicks in frequently, this might be a problem.

The interesting part is the *nativeStore* field that provides the COM+ seam to the Fabric internal interface. This is the interface that is closest to the Fabric surface and that allows to squeeze performance out of it.

### Summary

You can probably see, when this leads us. Seeing that underneath the .NET wrapper there is a COM+ that has much more low level interface and allows to use raw memory, we can try to access it directly, skipping *KeyValueStoreReplica* altogether and write a custom implementation that will enable to maximize the performance.
