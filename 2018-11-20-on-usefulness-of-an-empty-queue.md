---
layout: post
title: "On usefulness of an empty queue"
date: 2018-11-20 09:55
author: scooletz
permalink: /2018/11/20/on-usefulness-of-an-empty-queue/
image: /img/2018/11/x.png
categories: ["Azure", "serverless"]
tags: ["Azure", "serverless"]
whitebackgroundimage: true
nocomments: true
---

Recently, after running tests for a few weeks I published the first stable version of [QueueBatch](https://github.com/Scooletz/QueueBatch). QueueBatch is an Azure Function trigger that enables you to process Azure Storage Queue messages in batches in a really performant way. One of the recent additions was ability to run a function even if there are no messages in the queue. How could this be useful? Wouldn't it cost me money? Let me provide the answers below.

### Nothing to process

The first advantage that you might consider, for being notified on empty queue, is noticing an idle period of time. Having nothing to process means, that at least for now, when the call is made, there are no incoming messages. This time could be used for removing some old data, leasing blobs for some longer operations. The huge advantage over having an Azure Function timer trigger is following. When an empty queue is noticed, we know that the friction over leases or accessing different resources will be much smaller. In the next poll, there might be some messages, but at least for this single spin there are none.

### Costs, because it's all about money

If you use this "empty queue" notification for doing some work, are you paying more money paid for running your app? I'd dare to say that it's the same as you execute some kind of work that would need to be run anyway. You treat the empty queue just a special kind of a trigger.

### It's not idiomatic

One could argue that it's not an idiomatic use of a function. **Functions should serve a single purpose!** Yes, it's not idiomatic but useful, to progress with some work that requires less friction than usual operation. I'm leaving it up to the user what means *an idiomatic function usage* for an approach (FaSS) that is a few years old.

### Summary

Knowing that there's nothing to do can be useful. Spending this time on **this other thing** might help your application to progress with much smaller friction, in a much faster pace.

Oh, I can see, a new message batch arrived! Processing time!
