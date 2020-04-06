---
layout: post
title: "All the optimizations that you can't use"
date: 2019-07-22 08:55
author: scooletz
permalink: /2019/07/22/all-the-optimizations-that-you-cant-use/
image: /img/2019/07/opt-1.jpg
whitebackgroundimage: true
categories: ["C#", "Design"]
tags: ["dotnet", "serialization"]
---

*It's true. This blog post title is a bit sad. Still, it tries to show some cases that one needs to take into consideration when writing or designing a serializer. The article, as usual, assumes .NET environment.*

### Dynamic for everybody

The first obstacle are data passed in a form of *object*. Don't get me wrong, the top level of API call can be an *object*. What I mean by dynamic is having fields of object type inside of other objects.

```csharp
class MyData
{
    public object MoreData {get; set;}
}
```

This can happen, when a serializer is a generic purpose serializer and it's totally fine to expect a serializer to work on data like this. This is a bit limiting when comes to the optimizations though.

You cannot easily serialize data or inline the method for a field type as you need to check the type and dispatch it dynamically. Again, for a general purpose serializer it's a good property to have. Eventually, in JSON everything is an object, right?

### How big is it

Knowing the size of the serialized payload up-front is a really huge game changer. What you can do with this information? You can allocate the buffer before. That's the easiest one. What else?

You can ask a chunk of memory from a pool. In .NET, we've got the *MemoryPool* now, which can be asked to return *Memory{byte}*.

If the size of the needed memory is small enough and we're on the synchronous path (no async, no thread switching), we can also use *Span* with `stackalloc`.

```csharp
Span<byte> memory = stackalloc byte[maxEstimatedSize];
```

That's how [Enzyme, my experimental serializer works](https://github.com/scooletz/enzyme#use-stack-allocated-memory).

Again, if we have some really complex objects and going through the object tree is costly, it might be hard to go through this. If we add a not so strongly typed schema, and the need of checking types of objects and then visiting their payload on this basis. This is getting even more costly.

If you know the payload size, you can optimize for the memory usage. Unfortunately, for general purpose serializer, this might not be the case when a weakly-typed value is passed.

### General purpose vs system messages

That's the final question I presume. What do you design for? Is it a general purpose serializer, the one that should and will accept anything that it thrown into it, or are you designing a system protocol specific message? Or maybe there's some middle ground?

An example of a system specific message, with a rigid protocol could be:

1. NServiceBus [metrics message](https://github.com/Particular/ServiceControl.Monitoring.Data/blob/d1b15192315e041590e7a8d297b7b9b92afbd470/src/ServiceControl.Monitoring.Data/TaggedLongValueWriterV1.cs#L35-L47)
1. Hazelcast with [a wrapper around message payload](https://github.com/hazelcast/hazelcast-csharp-client/blob/c7546896caa6061c7f10500b190a51354551b47d/Hazelcast.Net/Hazelcast.Client.Protocol/ClientMessage.cs#L38-L54) which combines bits of general purpose (payload) and system message (wrapper)

### Summary

Designing a serializer requires you to make some decisions and choices. Even if you're writing a new JSON serializer, you'll need to choose what kind of types will be accepted, what kind of memory pooling (if any) can be used and how deep can we go to be specific (general purpose vs system). I hope this post provides you some of my experience in the serialization area.
