---
layout: post
title: "Memory models foundations"
date: 2016-03-03 11:00
author: scooletz
permalink: /2016/03/03/memory-models-foundations/
nocomments: true
categories: ["Optimization", "RampUp"]
tags: ["RampUpNet", "threading"]
imported: true
---

[RampUp](https://github.com/Scooletz/RampUp) is aimed at providing a very performant and aligned to nowadays hardware abstraction over modern CPUs. Although for using it one doesn't have to know all the corner cases of memory models, it's worth to know foundations supporting the RampUp library. This post touches the tip of the memory models iceberg. It's crucial to embrace that knowledge if one wants to write truly predictably performant code. And that's my aim in RampUp.

### Code is executed in a sequential way

It's a lie. It isn't executed in a sequential way. Considering the well known pyramid of latency, consider times spent on accessing

1. CPU registry/operation - 1ns
1. CPU cache - 10ns
1. DRAM access - 60ns
1. Disk, network - much more

If CPU waited just to get a value from DRAM before every operation, that would be highly inefficient. That's why caches were introduced. They are much closer to CPU and the look up takes much less. But you need to fill the cache. How is it done? In an optimal situation, CPU, before executing a given code chunk can evaluate all the needed addresses and order their prefetch to the CPU caches. Sometimes, if given 'lines of code' does not depend on each other, they may be reordered for later execution just to do not wait on the values being fetched from memory. This behavior depends on the CPU architecture, but if you want to sharpen your memory ordering related skills, I'm encouraging you to spend some time on considering these **reorderings**.

### Memory barriers

We can't live without order and neither can our code. The question is how much order do we need. If all the CPU instructions were ordered in a [strict order](http://mathworld.wolfram.com/StrictOrder.html) that would bring the starvation by waiting for the values being fetched from RAM. What are the ways of ensuring this ordering without destroying all the optimization possibilities connected with ordering. The mechanism for this is called memory barriers.

The most common one is the **full barrier**. This barrier creates a point that cannot be crossed by any operation: all the operations before this point will be executed before, all the operations located after - after. This type of barrier is issued for instance when:

* using *lock* statement - that's why when you use *locks* the behavior is so predictable
* using [Interlocked](https://msdn.microsoft.com/en-us/library/system.threading.interlocked) atomic operations like *Add*, *CompareExchange*. The operations are atomic but additionally, they introduce a full barrier (this applies to Intel x86/x64 processors)
* using [Thread.MemoryBarrier](https://msdn.microsoft.com/en-us/library/system.threading.thread.memorybarrier%28v=vs.110%29.aspx)
* using anything that uses two above (as operations are set in a particular order, so will the code using methods using above).

If there are full barriers, are there any other types? Yes, there are. We'll cover them in the next post.
