---
layout: post
title: "Hot paths in performance design"
date: 2018-09-24 08:55
author: scooletz
permalink: /2018/09/24/hot-paths-in-performance-design/
nocomments: true
image: /img/2018/09/path.jpg
categories: ["Design", "Uncategorized"]
tags: ["design", "performance"]
imported: true
---

The misconception that fast code must be ugly is unfortunately still alive. Also, there is an anti-pattern, of leaving the performance related work till the end, when nothing can be changed. Fortunately, there's a lot of people writing code that is fast and easy to follow. In this post I want to discuss hot paths, and how they are addressed in various APIs.

### Hot or not

One take on optimizations and designing for performance is taking into consideration the hot path of the execution. If a specific case covers 80, 90 or 99% (the higher the better) of the execution, optimizations introduced in there will likely pay off. I'm not saying that not-so-hot paths should be neglected. I'm noticing that recognizing the hot path and adding a single 'if' might change a lot. To make this happen, we need to provide some enablers, we need to think about this hot cases up front (no, sorry, it's disallowed to wave the dusty "premature optimization" flag here). Let's take a look at two examples from .NET world.

### ValueTask

When the new take on asynchronous programming was introduced in .NET, it took some time before it grew its roots. Nowadays, async-await can be seen in majority of the codebases bringing its performance benefits. The path towards *asyncification* was hard, but as the whole *you-need-to-be-async* settled down, .NET Core brought its new tool - *ValueTask*. ValueTask is a counterpart for the Task class and is a struct. If you have a possibly synchronous path in your async code, you can use it to return a value wrapped into an awaitable construct. Additionally, if you have a flow, that is guaranteed to do not issue too many async calls, you can use new API to pool objects implementing *IValueTaskSource* and cache its objects to make it even faster. What's the cost you could ask? It's barely none.

If you are a consumer with no intention for special handling the optimized case, you can just *await* for the call like you did in the past. It's possible. You can also follow the guidance of the returned type (*ValueTask*) and check if the call was returned in a synchronous flow. Then, with a single if, you can handle this case and speed up this part. If the synchronous flow is common (caching, prefetching) you can gain a lot. Still, a regular awaiting is available. What a great API it is, isn't it?

### Span, Memory and ReadOnlySequence

With new abstractions of Span and Memory provided for chunks of continuous memory, something might have been left unnoticed. It's quite common, that reading or writing a value won't be completed in one chunk. Sometimes, we need more (for example a big object being serialized) than we anticipated. In the old days, a stream abstraction was something that was used in this case. Nowadays, a construct that represents a chain of multiple *Memory* instances connected together, is handled by using *ReadOnlySequence*. You could say, that we're going back to the old days. It's another stream, but maybe a little better. It's a bit more explicit, similar to a linked list? What about cases where a single *Memory* is enough to write/read values? Can we optimize for it?

Unfortunately, for naysayers ;-) , we can address it quite easily. *ReadOnlySequence* provides two properties:

1. *IsSingleSegment*
1. *First*

that can be used to handle this special case. Does it clutter the regular case? No. Does it helps to handle a single segment case? For sure. Can I improve performance if the hot path is a single segment path? You bet.

### Summary

Designing performance for the hot path requires some enablers. If you build foundations (like BCL library or some core components for your system), it's good to think about various vectors and cases that they might be used in. Sometimes, enabling one hot path optimization does not clutter the design that much and can be a great enabler in the future.
