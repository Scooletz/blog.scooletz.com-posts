---
layout: post
title: "Replacing a generic dictionary"
date: 2016-03-30 09:00
author: scooletz
permalink: /2016/03/30/replacing-a-generic-dictionary/
nocomments: true
categories: ["Optimization", "RampUp"]
tags: ["optimization", "RampUpNet"]
imported: true
---

There is a moment, when you profile your high-throughput system and you hit the wall. And it's not your code but some BCL elements. That's what happened in [RampUp ](https://github.com/Scooletz/RampUp)when I was profiling *Write* part of the buffer.

The writer is emitted, but as the very foundation it uses a dictionary of metadata stored per message type. The metadata are simple:

* the message size
* the offset of the message envelope

Before optimization it was using the generic Dictionary specified with the message type and the message metadata. It was horribly slow. As the set of messages does not change, you could easily provide a set of message types up-front. For each message type one can obtain [RuntimeTypeHandle](https://msdn.microsoft.com/en-us/library/system.runtimetypehandle%28v=vs.110%29.aspx), which can be easily converted to *long*. With a set of longs, you could select minimal long and just subtract it from all the values. This would reduce the value to int. So here you are. You have a way of turning a type into int and ints are much easier to compare & to handle. One could even use a simple hash-map just to map between ints and metadata. This was the way to reduce the overhead of obtaining metadata with [IntLookup<TValue>](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Actors/Impl/IntLookup.cs). After applying the change, write performance has increased by 10%.
