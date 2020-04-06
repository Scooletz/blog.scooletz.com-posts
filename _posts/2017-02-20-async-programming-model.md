---
layout: post
title: "Async programming model"
date: 2017-02-20 09:55
author: scooletz
permalink: /2017/02/20/async-programming-model/
nocomments: true
image: /img/2017/02/stocksnap_6kas2j1v2a.jpg
categories: ["C#", "concurrency", "Polski"]
tags: ["async", "async await"]
imported: true
---

### TL;DR

This is a follow up post to [Async pump for better throughput in Azure](http://blog.scooletz.com/2017/02/02/async-pump-for-better-throughput-in-azure/). Please read the first before moving forward.

### Feedback

I've been given a lot of feedback about my *Async pump post*. In a few cases this [blog post from Ayende](https://ayende.com/blog/173473/fun-async-tricks-for-getting-better-performance) was quoted as it describes exactly the same approach. You can read the post, but more meaningful are comments provided by [Kelly Sommers](https://twitter.com/kellabyte "Comment by Kelly Sommers") and [Clemens Vasters](http://twitter.com/clemensv "Comment by Clemens Vasters").

### The model

The *await* statement has simple semantics. It breaks your code and schedules the following part as a task continuation. This heavy lifting is done on the C# compiler level, so you don't have to worry about. The model of this extension is simple: define a task and a continuation. Nothing more, nothing less. With my approach, that was a "trick"

> That's the premise of the "trick" that is allegedly achieving parallel execution of I/O and compute work here. That is, however, not the purpose of the asynchronous programming model and of the Windows IO completion port (IOCP) model. The point of IOCP is to efficiently offload IO work from user code to kernel and driver and hardware and not to bother the user code until the IO work is done

by Clemens Vasters

What it basically says is once you await on IO operation, your code that is run after, is scheduled on the IO thread

> As IO typically takes very long and compute work is comparatively cheap, the goal of the IO system is to keep the thread count low (ideally one per core) and schedule all callbacks (and thus execution of interleaved user code) on that one thread. That means that, ideally, all work gets serialized and there minimal context switching as the OS scheduler owns the thread. The point of this model is ALREADY to keep the CPU nicely busy with parallel work while there is pending IO.

by Clemens Vasters

So with your code awaiting some IO, you'll call IO, next the code after *await* is executed on the IO thread, as it's assumed to be lightweight, another IO occurs, again the same thread dispatched the callback. With the *async* pump I proposed, comes a danger, as when we follow this partially *awaitless* approach the continuation code is

> not on a .NET poll thread, but an IO pool thread. The strategy above sits on that IO pool thread past the point where you are supposed to return it (which is the next IO call) and thus you might force the IO pool to grow

by Clemens Vasters

### Summary

Measure first. Think. Then think again. The "trick" worked in my case, improving performance for initialization of one app (I needed it in the beginning). Does it work in every case and should be used in general, I'd say no. It's good to follow the programming model. If you don't want to, you must have strong reasons for it.
