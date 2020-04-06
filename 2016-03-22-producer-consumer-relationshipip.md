---
layout: post
title: "Producer - consumer relationship"
date: 2016-03-22 09:17
author: scooletz
permalink: /2016/03/22/producer-consumer-relationshipip/
nocomments: true
categories: ["Design", "Design patterns", "RampUp"]
tags: ["concurrency", "design patterns", "RampUpNet"]
imported: true
---

In [the last post](http://blog.scooletz.com/2016/03/18/unsafe-buffer-in-rampup/) about the [RampUp](https://github.com/Scooletz/RampUp) library I covered on of the foundations: *IRingBuffer*. Now I'd like to describe the contract it fulfills.

### Producer consumer

If you take a look at the [IRingBuffer](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/IRingBuffer.cs) you'll see Write/Read methods. These two are responsible for producing/consuming or writing/reading messages to the buffer in FIFO way. What are the guarantees behind such an interface? What about concurrent accessing this structure?

### Multimulti

The easiest approach for distinguishing accessing patterns is considering whether the structure could be accessed by one or more threads. If you consider producer/consumer you'll see that there are four options:

1. SPSC - Single Producer Single Consumer - only one thread produces items, and another consumes them
1. MPSC - Multi Producer Single Consumer - multiple threads may produce items in a safe manner, again there's a single consuming thread
1. SPMC - Single Producer Multi Consumer - this could be treated as a distributor of work
1. MPMC - Multi Producer Multi Consumer - multi/multi, [ConcurrentQueue](https://msdn.microsoft.com/en-us/library/dd267265%28v=vs.110%29.aspx) is a good example of it

Unfortunately nothing comes for free. If you want to get **multi**, you'll pay the price of handling friction on that end. On the other hand, if you want to design a system where queues provide transport between different parts of the system, you'll need to enable **multiple producers** for sure, as there's going to be more than one system element. If you want to process items in an order they appeared and leave the locking issues, just write a fast single threaded code, **a single consumer** with a single worker thread would be the way to go. Of course this worker thread may access other queues and produce items for them (hence, multi producer is needed).

The [ring buffer implementation](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/ManyToOneRingBuffer.cs) in RampUp provides exactly MPSC behavior, as it's prepared to handle items in order, by a single thread.
