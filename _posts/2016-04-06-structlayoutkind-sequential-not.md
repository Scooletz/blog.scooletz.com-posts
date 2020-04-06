---
layout: post
title: "StructLayoutKind.Sequential not"
date: 2016-04-06 10:00
author: scooletz
permalink: /2016/04/06/structlayoutkind-sequential-not/
categories: ["Optimization", "RampUp"]
tags: ["memory", "memory layout", "performance", "RampUpNet"]
imported: true
---

If you want to write a performant multi threaded application which actually is an aim of [RampUp](https://github.com/Scooletz/RampUp), you have to deal with padding. The gains can be pretty big, considering that the whole work with threads mean, that you need to give them their own spaces to work in.

### False sharing

[False sharing](https://en.wikipedia.org/wiki/False_sharing) is nothing more than two or more threads trying to use memory that's mapped to a single line of cache. The best case for any thread is to have their own memory space separated & by separation I mean having enough of padding on the right and on the left, to keep the spaces of two threads without any overlapping. The easiest way is to add additional 64 bytes (the size of a cache line) at the end and at the beginning of the struct/class to ensure that no other thread will be able to allocate memory close enough. This mechanism is called padding.

### Padding

The easiest way to apply padding is applying [StructLayoutAttribute.](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.structlayoutattribute%28v=vs.110%29.aspx) If *StructLayoutKind.Sequential* is used, then adding 4 Guid fields at the beginning and 4 Guid fields at the end should work just fine. The size of Guid is 16 bytes which give us needed 64 bytes. A harder way of doing it is using *StructLayoutKind.Explicit* as it requires to add [FieldOffsetAttribute](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.fieldoffsetattribute%28v=vs.110%29.aspx) to every field of the structure/class, explicitly stating the offset in the memory. With this approach, it's easy to start with 64 and leave some space at the end of the class.

### Problem

*StructLayoutKind.Sequential* works perfectly. Almost. Unfortunately if any field has type that is not *Sequential* or *Explicit* CLR will simply ignore the sequential requirement and silently apply automatic layout ruining the padding. This is a regular case, all classes use *Auto* by default. Unfortunately it leaves the developer with the need of applying the fields offsets manually.

### Solution

As I need this padding behavior for RampUp, I'm creating a small [Fody weaver](https://github.com/Fody/Fody) plugin called [Padded ](https://github.com/Scooletz/Padded)which will automatically calculate offsets (possibly with some memory overhead) for any class/struct marked with a proper attribute. Hopefully, it will be useful not only for RampUp but for more, performance oriented projects
