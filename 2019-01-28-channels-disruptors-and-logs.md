---
layout: post
title: "Channels, ring buffers and logs"
date: 2019-01-28 09:55
author: scooletz
permalink: /2019/01/28/channels-disruptors-and-logs/
image: /img/2019/01/channels.png
categories: ["async", "dotnet", "performance"]
tags: ["async", "dotnet", "performance"]
whitebackgroundimage: true
---

If you're developing applications in .NET, you probably heard about all the new shiny part of the framework, like Pipelines which enable you to process IO-related processing with more IO awareness, still leaving your code on quite high level. Another part of the framework that is mentioned recently are channels that are used to pass data between parties. In this post I discuss various approaches used for data passing.

## Channels

At the beginning it's worth to define what *Channels* are. They are provided in *System.Threading.Channels* namespaces. They are not that big, you can take the look at [their implementation on GitHub](https://github.com/dotnet/corefx/tree/master/src/System.Threading.Channels/src/System/Threading/Channels). What they do is they provide a way to communicate between tasks/threads, offering *async* methods for cases, where there is no data/space on one end of the channel. With *async* methods a publishing/consuming party can just schedule a continuation (in other words: **await**) the condition for proceeding.

To start with channels one needs to create one. To do it, one of the following factory methods should be used:

```csharp
public static class Channel
{
  public static Channel CreateUnbounded()
  {
    // ...
  }

  public static Channel CreateUnbounded(
    UnboundedChannelOptions options)
  {
    // ...
  }

  public static Channel CreateBounded(int capacity)
  {
    // ...
  }

  public static Channel CreateBounded(
    BoundedChannelOptions options)
  {
    // ...
  }
}
```

### Boundness

Here, you can notice two categories of channels that you can create:

1. bounded - they have a limited size. Good for scenarios when you know that number of items in a channel won't breach a small specified size
1. unbounded - not limited in size. Growing and shrinking as needed according to the pressure

### Concurrency

The other parameter is the number of customers and producers than can be configured via channel options.

```csharp

public abstract class ChannelOptions
{
  public bool SingleWriter { get; set; }

  public bool SingleReader { get; set; }

  public bool AllowSynchronousContinuations { get; set; }
}

```

This is a standard parameter for data structures.  They can be:

1. **S**ingle **P**roducer - enabling only a single thread/task to append to the structure
1. **M**ulti **P**roducer - enabling many threads/tasks to append to the structure at the same time
1. **S**ingle **C**onsumer - enabling only a single thread/task to consume from the structure
1. **M**ulti **C**onsumer - enabling many threads/tasks to consume from the structure

These two categories can be combined, so there can be for example **MPSC** (multi producer single consumer). Just to give a real world example, *ConcurrentQueue* represents **MPMC**. The reason for differentiating a single from many are optimizations that can be applied whenever there is a single party acting on its end of the channels. The approaches that one can take to optimize for, are beyond of the scope of this article though (if you are interested though, I encourage you to spend a week at [1024cores](http://www.1024cores.net), one of my favorite sites describing concurrent data structures).

### Data passing and implementation

In *Channels* data passing is realized by copying the value. If the channel is created for *T* which is a *ValueType* the value will be copied. If *T* is a reference type, it will work by passing (copying) the reference to the object. It's exactly the same behavior as in *ConcurrentQueue* and all the other concurrent data structures. There's no way to pass large amounts of data, if you don't manage them manually (pooling memory, etc.).

The mention of *ConcurrentQueue* in the paragraph above was intentional. The *UnboundedChannel* is just a simple wrap around the ConcurrentQueue adding some synchronization for the cases, where there is no data for the reader. Just take a look, [the implementation is less than 400 lines](https://github.com/dotnet/corefx/blob/master/src/System.Threading.Channels/src/System/Threading/Channels/UnboundedChannel.cs).

Having Channels based on an already proven and improved in .NET Core concurrent collection, hopefully will make them better every single time ConcurrentQueue is upgraded (in .NET Core it has been upgraded a lot).

## Disruptor

Another approach for data passing is using a preallocated buffer (a ring buffer) and reusing array cells for producers and consumers. This approach was greatly re-introduced to a managed world by [Disruptor project](https://github.com/LMAX-Exchange/disruptor) from LMAX, a high-frequency trading company. There's a dotnet port of it, called [Disruptor-net](https://github.com/disruptor-net/Disruptor-net).

Disruptor works on a single array of values, treating it as a continuous never-ending buffer. Whether or not a cell can be consumed or written to is defined by calculating the gating values. If there's only a single producer and a single consumer, only two gating values are needed, let's call them P and C. Additionally, we assume Length as the length of the array.

1. P - index where producer will try to write to
1. C - index where consumer will try to read from
1. Length - lengths of the array

If the following holds true

> P - C > Length

, in other words, there's a space in the buffer, producer can write to a cell, advancing P by one. Once the end of the buffer is met, we start indexing from the beginning, hence, the **circular buffer** name.

It's interesting that with this approach, one can introduce multiple consumers chained one after another, let's call them C1, C2, C3, C4... In terms of the numbers, still they can be ordered in the following way, from the highest, to the lowest

> P, C1, C2, C3

The initial condition for the publisher, would be checked against the last CN (as all the rest is somewhere in the middle).

### By ref

One of the biggest advantages of this approach is having a preallocated array and operating on existing data. To make it simpler, we could think of it as a producer writing to the following ref returned value

```csharp
return ref buffer[index];
```

With this approach, even for values spanning several fields there's no copying as the value is written directly to the buffer. Of course, additionally, it needs to be communicated when the write is done, but this is a separate story. In disruptor, the gating value is incremeted to let know the next element of the chain to move forward.

Unfortunately, this approach lacks the support for the *async* path. One could imagine though trying to combine this approach with async support just to enable passing tuples consisting of several values. It's interesting that under the hood, *ConcurrentQueue* uses a very similar concept reusing a buffer when the growth is not needed.

## Logging like Aeron

The last but not least is [Aeron](https://github.com/real-logic/aeron), a powerful messaging over UDP. It's based on [Agrona](https://github.com/real-logic/agrona/), a Java library containing many concurrent data structures. One of its most interesting ones is the [ManyToOneRingBuffer](https://github.com/real-logic/agrona/blob/master/agrona/src/main/java/org/agrona/concurrent/ringbuffer/ManyToOneRingBuffer.java), enabling efficient data producing from many publishers to a single customer (it's **MPSC**).

The approach taken by Agrona/Aeron is based on value passing. Whenever one want to write to the buffer, they need to serialize the data and write it as a message. With this approach, and ensuring a proper ordering of operations (a few *volatile-*s used), Agrona enables writing values concurrently. The consumer can process a value only when it's written fully by its producer. This approach costs a bit on the writing side (value needs to be serialized) but the consumer can easily swallow batches of messages. Aeron uses this to create UDP frames and send them efficiently over the network. If you're interested in this approach, read my [smart batching article](http://blog.scooletz.com/2018/01/22/the-batch-is-dead-long-live-the-smart-batch/) .

Again, Aeron does not provide *async* path. Even with its bounded buffer, allocating sufficiently big buffer should address the need of waiting for the space to write.

## Summary

In this article I compared several ways of cross thread/task communication. They all favor one thing over another, bringing value for the specific problem.

I really like *Channels* getting to the dotnet core. It shows that pushing forward with good foundations for cross thread/task communication is important for the dotnet team. I'm looking forward for all the next big steps.
