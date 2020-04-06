---
layout: post
title: "Epoch based programming"
date: 2018-08-20 08:55
author: scooletz
permalink: /2018/08/20/epoch-based-programming/
nocomments: true
image: /img/2018/08/epochs.jpg
categories: ["Distributed systems", "Optimization"]
tags: ["concurrency", "distributed"]
imported: true
---

Human beings like categorizing things. Categories make things simpler, much easier to grasp. The same is applied to events when we assign them to a specific epoch or a period of our history. Can we use epochs in computer systems and programming? Would they be useful? Would they make things more complex or simpler? Let's discuss it.

### A brief history of time

Before we immerse into computer systems, let me sketch a useful definition of an epoch:

1. epochs do not overlap
1. every epoch has a predecessor (beside the first one)
1. epochs are numbered in a monotonically increasing way (1, 2, 5, 6)

As you can see, this is not a strict mathematical definition. This is just a sketch that will help us grasp our intuition in a bit more strict way. So, as we know that epochs are nothing more than numbered buckets, we can move on to taking a look at different areas of programming, in which they are used.

### Consensus algorithm

One of the cardinal problems of distributed systems is reaching for the consensus. The consensus is often rephrased as a replicated state machine problem. If we are able to replicate all the operations of a state machines in the same order to multiple nodes, these nodes, applying the same algorithm would end in the same state. Now, you could ask what epochs have to do with it? Let me show you.

One of foundations of consensus algorithms is a notion of leader. A leader is a node, that is somehow selected (elected) from all the available nodes, that all the writes go through. Once, a leader fails, another one is elected. But how do we know, which leader is it? In other words, how to separate leader from this moment, from a leader from the past? Epochs come to the rescue!

Whenever a leader fails, all the nodes bump up their epoch number and issue another voting. Once a leader is elected, it's a leader in the specific epoch! No two leaders can be elected in the same epoch. This, with addition of epochs moving forward, allows some algorithms to resolve various problems in an efficient and easy way.

If this made you interested in this topic, I cannot encourage you more to take a look at [Raft algorithm](https://raft.github.io/) and (if you're brave enough) [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science)), which uses ballot numbers (a slightly different take on epochs).

### Faster than...

Recently, a group from Microsoft Research, has published a paper about *Faster*. Faster is a database, based on understanding of modern hardware, that

> combines combines a highly cache-optimized concurrent hash index with a hybrid log: a concurrent log-structured record store that spans main memory and storage, while supporting fast in-place updates of the hot set in memory.

One of the most important foundations of this database is usage of epochs. Each thread that is used by the application, has its own bucket for storing the recently observed epoch number. The thread might have a stale value, for instance equal to 2, when the current epoch is equal to 4. Having a stale value is perfectly fine. The more important aspect is that for all the threads we might define the lower and the upper boundary for an epoch (just min and max). For threads holding epochs: 1, 2, 3, 5, the lower boundary would be 1, the upper - 5. This makes it possible to ensure that all the threads moved over a specific epoch and some data can be flushed/stored/blocked for updates. The current epoch is bumped up by any of the threads. At the same time, threads will read the current epoch from time to time updating their own copy, which eventually, will move forward the lower boundary.

If this usage of epochs made you eager to learn more about Faster, this is [the page](https://www.microsoft.com/en-us/research/project/faster/) you should take a look at.

### Lock-free/wait-free algorithms and visibility

If you read and (hopefully) write fast code, for sure you meet some *volatiles* from time to time. It might be *Unsafe* in Java, *Interlocked/Volatile* in C# or even *<atomic>* in C++*.* All these constructs allow writing code, that will store values in a way, that is eventually visible by the reader. One of the techniques, that you could use to notify other agents in your process about a change of a value is using an increasing epoch-like number.

Let's assume that you want to write a value to an array that can be consumed by another agent. You could write the value first (in a non-atomic way, with no locks), and confirm it by assigning a sequence field to sequence + 1. Something like in the C# code below.

```csharp
// some checks here
data[producer].Value1 = something1;
data[producer].Value2 = something2;
Volatile.Write(ref data[producer].Sequence, producer + 1);

```
On the consumer side:

```csharp

// some checks here
DoSomethingWithData(ref data[consumer]);
Volatile.Write(ref data[consumer].Sequence, consumer + data.Length);

```

With this approach, and additional checks (some ifs before these method bodies) with a *Volatile.Read*, the *Sequence* number is like an epoch: always increasing and allowing only one of the sides to move forward.

If you're interested into this low-level kind of programming, especially using lock-free/wait-free techniques, take a look at [1024 cores' implementation of the concurrent queue](http://www.1024cores.net/home/lock-free-algorithms/queues/bounded-mpmc-queue). The whole site is filled with very interesting, in-depth material, that you can use to study these categories of algorithms.

### Summary

An epoch-based approach to different aspects of systems, make it really easy to reason about. A simple natural number moving forward, assigned to every operation, shows when and who done it. I hope this will encourage you to search for and learn about this powerful simplicity of epochs.
