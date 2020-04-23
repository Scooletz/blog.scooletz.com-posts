---
layout: post
title: "The batch is dead, long live the smart batch"
date: 2018-01-22 09:55
author: scooletz
permalink: /2018/01/22/the-batch-is-dead-long-live-the-smart-batch/
image: /img/2018/01/channels.png
categories: ["architecture", "design"]
tags: ["architecture", "design"]
whitebackgroundimage: true  
nocomments: true
---

It lurks in the night. It consumes all the energy. It lasts much too long. If you experienced it, you know it's unforgettable. If you're lucky and you did not meet it, you probably heard these stories from your friends. Yes, I'm talking about the **batch job**.

### The old batch

This ultimate tool of terror was dreading us for much too long. Statements like "let's wait till tomorrow" or "I think that the job didn't run" were quite popular a few years back. Fortunately for us, with the new waves of reactive programming, serverless and good old-fashioned queues, it's becoming thing of the past. We're in a very happy position being able to process events, messages, items as soon as they come into our system. Sometimes a temporary spike can be amortized by a queue. And it works. Until it's not.

When working on processing **2 billion events per day** with Azure functions, I deliberately started with the assumption of 1-1 mapping, where one event was mapped as one message. This didn't go well (as planned). Processing 2 billion items can cost you a lot, even, if you run this processing on-premises, that are frequently treated as "free lunch". The solution was easy and required going back to the original meaning of the batch, which is a group, a pack. The very same solution that can be seen in so many modern approaches. It was **smart batching**.

### The smart batch

If you think about regular batching, it's unbounded. There's no limit on the size of the batch. It must be processed as a whole. Next day, another one will arrive. The smart batching is different. It's meant to batch a few items in a pack, just to amortize different costs of:

1. storage (accessing store once)
1. transport (sending one package rather than 10; I'm aware of the Nagle's algorithm)
1. serialization (serializing an array once)

To use it you need:

1. a buffer (potentially reusable)
1. a timer
1. potentially, an external trigger

It works in the following way. The buffer is a concurrent-friendly structure that allows appending new items by, potentially, multiple writers. Once

1. the buffer is full or
1. the timer fires or
1. the external trigger fires

all the items that are in the buffer will be sent to the other party. This ensures that the flushing has:

1. a bounded size (the size of the buffer)
1. a bounded time of waiting for ack (the timer's timeout)

With this approach, used actually by so many libraries and frameworks, one could easily overcome many performance related issues. It amortizes all the mentioned costs above, paying a bit higher tax, but only once. Not for every single item.

### Outsmart your costs

The smart batch pattern enables you to cut lots of costs. For cloud deployments, doing less IO requests means less money spent on the storage. It works miracles for the throughput as well, no wonder that some cloud vendors allow you to obtain messages in batches etc. It's just better.

Next time when you think about a batch, please, do it,  but in a smart way.
