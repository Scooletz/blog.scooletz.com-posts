---
layout: post
title: "Single producer single consumer optimizations"
date: 2016-05-16 09:00
author: scooletz
permalink: /2016/05/16/single-producer-single-consumer-optimizations/
nocomments: true
categories: ["RampUp", "Uncategorized"]
tags: ["RampUpNet", "Volatile"]
imported: true
---

The producer-consumer relationship is one of the most fundamental cooperation patterns. Some components produce values, issues requests and some consume/handle them. Depending on the number of components at the end of this dependency it's called 'single/multi producer single/multi consumer' relationship. It's important to make this choice explicit, because as with every explicit choice, it enables some optimizations. I'd like to share some thoughts o the optimizations taken in the single consumer single producer scenario in the [RampUp](https://github.com/Scooletz/RampUp) library provided by [OneToOneRingBuffer](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/OneToOneRingBuffer.cs).

The behavior of ring buffers in RampUp is ported from [Java's Agrona](https://github.com/real-logic/Agrona). They provide a queue that enables reading sequentially on the consumer side. The reasoning behind it is that sequential reads are CPU friendly, so that consumer can process messages much quicker. For [ManyToOneRingBuffer ](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/ManyToOneRingBuffer.cs)the production part is quite complex. It proceeds as follows:

1. check against the consumer position, is there enough of space
1. allocate a slot in the ring (this is done with Interlocked operations, in a loop, may take a while)
1. write a header in an ordered way (using volatile)
1. put data
1. write the header again marking the message as published

This brings a lot of unneeded work for a single producer. When considering a single producer, there's nothing to compete with. The only check that needs to be made is that the producer does not overlap with the consumer. So the algorithm looks as follows:



1. check against the consumer position, is there enough of space
1. put data
1. write the header again marking the message as published
1. write the tail value for future writes

Removal of Interlocked and lowering the number of Volatile operations can improve the producer performance greatly (less synchronization).



If you wanted to compare these two on your own, here you are: [ManyToOne ](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/ManyToOneRingBuffer.cs#L52)and [OneToOne](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Ring/OneToOneRingBuffer.cs#L101).

Happy producing (and consuming).
