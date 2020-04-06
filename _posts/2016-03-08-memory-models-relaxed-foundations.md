---
layout: post
title: "Memory models relaxed foundations"
date: 2016-03-08 11:00
author: scooletz
permalink: /2016/03/08/memory-models-relaxed-foundations/
nocomments: true
categories: ["Optimization", "RampUp"]
tags: ["RampUpNet", "threading"]
imported: true
---

In [the previous post](http://blog.scooletz.com/2016/03/03/memory-models-foundations/) we've built up some basic foundations for total ordering of CPU instructions. It was mentioned that this total ordering is much too strict and if we want to use the hardware in the best possible way, there's a need of relaxed barriers, that allow some reorderings, especially when considering high-througput libraries like [RampUp](https://github.com/Scooletz/RampUp).

### 2x2 = 4

From the CPU perspective there are only two operations that can be performed on a memory location. These are:

* Store - a value is read from the memory
* Load - a value is written to the memory

If you consider an order of two operations you can get following chains (pairs):

* Load-Store
* Load-Load
* Store-Load
* Store-Store

These for pairs are enough to consider different reorderings. One can easily imagine reordering elements in each of these pairs. Knowing these pairs and the definition of the full memory barrier, one can easily reason, that the full barrier prohibits all of the reorderings mentioned above.

### Volatile this, volatile that

There's a class in .NET, used very infrequently, called [Volatile](https://msdn.microsoft.com/en-us/library/system.threading.volatile). It has only two methods, with many overloads. They're *Read* and *Write*. Here's their semantics:
* Volatile.Read - is equal to the following sequence of operations:

    *   read the value from the memory

    *   issue a barrier prohibiting Load-Store & Load-Load reorderings
* Volatile.Write - is equal to the following sequence of operations:

    *   issue a barrier prohibiting Store-Store & Load-Store reorderings

    *   write the value to the memory

These two methods should be thought of as siblings and should be used together. If one thread writes value with a Volatile.Write & the other reads with Volatile.Read, the value written by the first will be visible to the second after a while, in other words, reading the value in the loop finally will result in the value written by the first thread. This behavior connected with the barriers and disabling some of the reorderings may create an extremely performant approaches.

### Summing up & RampUp implications

Now, the memory barriers, reorderings are known a bit better. You know, that an efficient & performant code is a code that lets a CPU for reorderings to optimize the pipeline. You know also, that beyond full memory barriers there are much less obstructive methods that can be used to apply partial ordering of instructions. As our knowledge about memory expands, we're getting closer to analyze the very first element of the RampUp, [AtomicLong](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Atomics/AtomicLong.cs) & [AtomicInt.](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Atomics/AtomicInt.cs)
