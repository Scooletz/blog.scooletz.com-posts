---
layout: post
title: "Much Better Stateful Service Fabric"
date: 2017-04-10 08:55
author: scooletz
permalink: /2017/04/10/much-better-stateful-service-fabric/
nocomments: true
image: /img/2017/03/scooletz_image_better_2.jpg
categories: ["Azure", "Service Fabric", "Sewing Machine"]
tags: ["azure"]
imported: true
---

### TL;DR

In the last post we found the [COM+ interface that is a foundation](http://blog.scooletz.com/2017/04/06/better-stateful-service-fabric) for *KeyValueStoreReplica* persistence. It's time to describe the way, how the underlying performance can be unleashed for the managed .NET world.

### Internal or not, I don't care

The sad part of this COM+ layer of ServiceFabric is that's internal. The good part of .NET is that when running in Full Trust, you can easily overcome this obstacle with *Reflection.Emit* namespace and emitting helper methods wrapping internal types. After all, there is a public surface that you can start with. The more you know about MSIL and internals of CLR and the more you love it, the less tears and pain will be caused by the following snippet code. Sorry, it's time for some IL madness.

```csharp

var nonNullResult = il.DefineLabel();

// if (result == null)
//{
//    return null;
//}
il.Emit(OpCodes.Ldloc_0);
il.Emit(OpCodes.Brtrue_S, nonNullResult);
il.Emit(OpCodes.Ldnull);
il.Emit(OpCodes.Ret);

il.MarkLabel(nonNullResult);

// GC.KeepAlive(result);
il.Emit(OpCodes.Ldloc_0);
il.EmitCall(OpCodes.Call, typeof(GC).GetMethod("KeepAlive"), null);

// nativeItemResult.get_Item()
il.Emit(OpCodes.Ldloc_0);
il.EmitCall(OpCodes.Callvirt, InternalFabric.KeyValueStoreItemResultType.GetMethod("get_Item"), null);

il.EmitCall(OpCodes.Call, ReflectionHelpers.CastIntPtrToVoidPtr, null);
il.Emit(OpCodes.Stloc_1);

// empty stack, processing metadata
il.Emit(OpCodes.Ldloc_1);   // NativeTypes.FABRIC_KEY_VALUE_STORE_ITEM*
il.Emit(OpCodes.Ldfld, InternalFabric.KeyValueStoreItemType.GetField("Metadata")); // IntPtr
il.EmitCall(OpCodes.Call, ReflectionHelpers.CastIntPtrToVoidPtr, null); // void*
il.Emit(OpCodes.Stloc_2);

il.Emit(OpCodes.Ldloc_2); // NativeTypes.FABRIC_KEY_VALUE_STORE_ITEM_METADATA*
il.Emit(OpCodes.Ldfld, InternalFabric.KeyValueStoreItemMetadataType.GetField("Key")); // IntPtr

il.Emit(OpCodes.Ldloc_2); // IntPtr, NativeTypes.FABRIC_KEY_VALUE_STORE_ITEM_METADATA*
il.Emit(OpCodes.Ldfld, InternalFabric.KeyValueStoreItemMetadataType.GetField("ValueSizeInBytes")); // IntPtr, int

il.Emit(OpCodes.Ldloc_2); // IntPtr, int, NativeTypes.FABRIC_KEY_VALUE_STORE_ITEM_METADATA*
il.Emit(OpCodes.Ldfld, InternalFabric.KeyValueStoreItemMetadataType.GetField("SequenceNumber")); // IntPtr, int, long

il.Emit(OpCodes.Ldloc_1); // IntPtr, int, long, NativeTypes.FABRIC_KEY_VALUE_STORE_ITEM*
il.Emit(OpCodes.Ldfld, InternalFabric.KeyValueStoreItemType.GetField("Value")); // IntPtr (char*), int, long, IntPtr (byte*)

```

The part above is just a small snippet responsible partially for reading the value. If you're interested in more, here are [over 200 lines of emit](https://github.com/Scooletz/SewingMachine/blob/master/src/SewingMachine/RawAccessorToKeyValueStoreReplica.cs#L100-L324) that brings the COM+ to the public surface. You don't need to read it though. SewingMachine delivers a much nicer interface for it.

### <span class="pl-en">RawAccessorToKeyValueStoreReplica</span>

[<span class="pl-en">RawAccessorToKeyValueStoreReplica</span>](https://github.com/Scooletz/SewingMachine/blob/master/src/SewingMachine/RawAccessorToKeyValueStoreReplica.cs) is a new high level API provided by SewingMachine. It's not as high level as the original *KeyValueStoreReplica* as it accepts *IntPtr* parameters, but still, it removes a lot of layers leaving the performance, serialization, memory management decisions to the end user of the library. You can use your own serializer, you can use *stackalloc* to allocate on the stack (if values are small) and much much more. This accessor is a foundation for another feature provided by SewingMachine, called *KeyValueStatefulService*, a new base class for your stateful services.

### Summary

We saw how *KeyValueStoreReplica* is implemented. We took a look at the COM+ interface call sites. Finally, we observed how, by emitting IL, one can expose an internal interface, wrap it in a better abstraction and expose it to the caller. It's time to take a look at the new stateful service.
