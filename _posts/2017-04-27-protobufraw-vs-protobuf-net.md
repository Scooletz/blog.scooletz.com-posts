---
layout: post
title: "ProtobufRaw vs protobuf-net"
date: 2017-04-27 08:55
author: scooletz
permalink: /2017/04/27/protobufraw-vs-protobuf-net/
nocomments: true
categories: ["Profiling", "Protobuf-net", "Service Fabric", "Sewing Machine"]
tags: ["BenchmarkDotNet", "performance"]
imported: true
---

### TL;DR

I'm working currently on [SewingMachine](https://github.com/Scooletz/SewingMachine), an OSS project of mine, that is aimed at unleashing the ultimate performance for your stateful services written in/for Service Fabric (more posts: [here](https://blog.scooletz.com/category/sewing-machine/)). In this post I'm testing further (previous test is [here](http://blog.scooletz.com/2017/04/20/threadlocal-vs-stackalloc)) whether it would be beneficial to write a custom unmanaged writer for protobuf-net using *stackallock*ed memory.

### SewingMachine is raw, very raw

SewingMachine works with pointers. When storing data, you pass an *IntPtr* with a length as a value. Effectively, it means that if you use a managed structure to serialize your data, finally you'll need to either pin it (pinning is notification for GC to do not move object around when it's pinned) or have it pinned from the very beginning (this approach could be beneficial if an object is large and has a long lifetime). If you don't want to use managed memory, you could always use *stackalloc* to allocate a small amount of memory on stack, serialize to it, and then pass it as IntPtr. This is approach I'm testing now.

### Small, fixed sized payloads

If a payload, whether it's an event or a message is small and contains no fields of variable length (strings, arrays) you could estimate the maximum size it will take to get serialized. Next, instead of using Protobuf-net regular serializer, you could write (or emit during a post-compilation) a custom function to serialize a given type, like I did in this [spike](https://gist.github.com/Scooletz/851263e9272ae2d95b1e39a8439dddac). Then it's time to test it.

### Performance tests and over 10x improvement

Again, as in the previous post about memory, the unsafe stackallock version shows that it could be beneficial to invest some more time as the performance benefit is just amazing . The raw version is 10x faster. Wow!

![protoraw_vs_proto](/img/2017/04/protoraw_vs_proto.png)

### Summary

Sometimes using raw unsafe code improves performance. It's worth to try it, especially in situations where the interface you communicate with is already unsafe and requiring to use unsafe structures.
