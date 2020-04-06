---
layout: post
title: "ThreadStatic vs stackalloc"
date: 2017-04-20 08:55
author: scooletz
permalink: /2017/04/20/threadlocal-vs-stackalloc/
nocomments: true
categories: ["Profiling", "Protobuf-net", "Service Fabric", "Sewing Machine"]
tags: ["BenchmarkDotNet", "performance"]
imported: true
---

### TL;DR

I'm working currently on [SewingMachine](https://github.com/Scooletz/SewingMachine), an OSS project of mine, that is aimed at unleashing the ultimate performance for your stateful services written in/for Service Fabric (more posts: [here](https://blog.scooletz.com/category/sewing-machine/)). In this post I'm testing whether it would be beneficial to write a custom unmanaged writer for protobuf-net, instead of using some kind of object pooling with ThreadLocal.

### ThreadStatic and stackalloc

[ThreadStatic](https://msdn.microsoft.com/en-us/library/system.threadstaticattribute(v=vs.110).aspx) is the old black. It was good to use before async-await has been introduced. Now, when you don't know on which thread your continuation will be run, it's not that useful. Still, if you're on a good old-fashioned synchronous path, it might be used for object pooling and keeping one object per thread. That's how [protobuf-net caches ProtoReader objects](https://github.com/mgravell/protobuf-net/blob/master/src/protobuf-net/ProtoReader.cs#L1362-L1378).

One could use it to cache locally a chunk of memory for serialization. This could be a managed or unmanaged chunk, but eventually, it would be used to pass data to some storage (in my case, [SewingSession from SewingMachine](https://github.com/Scooletz/SewingMachine/blob/master/src/SewingMachine/SewingSession.cs)). If the interface accepted unmanaged chunks, I could also use *stackalloc* for small objects, that I know how much memory will be occupied by. *stackalloc* provides a way to allocate some number of bytes from the stackframe. Yes, it's unsafe so keep your belts fastened.

### ThreadStatic vs stackalloc

I gave it a try and wrote a simple (if it's dummy, I encourage you to share your thoughts in comments) test that either uses a ThreadStatic-pooled object with an array or a *stackalloc*ated and writes. You can find it in [this gist](https://gist.github.com/Scooletz/2aeee833894e8239bfdb6d06e036118d).

How to test it? As always, to the rescue comes [BenchmarkDotNet](http://benchmarkdotnet.org/), the best benchmarking tool for any .NET dev. Let's take a look at the summary now.

![local_vs_threadstatic.png](/img/2017/04/local_vs_threadstatic.png)

Stackalloc wins.

There are several things that should be taken into consideration. Finally block, the real overhead of writing an object and so on and so forth. Still, it looks that for heavily optimized code and small objects, one could this to write them a bit faster.

### Summary

Using *stackalloc*ated buffers is fun and can bring some performance benefits. If I find anything unusual or worth noticing with this approach, I'll share my findings. As always, when working on performance, measure first, measure in the middle and at the end.
