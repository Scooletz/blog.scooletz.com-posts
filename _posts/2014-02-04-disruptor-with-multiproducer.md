---
layout: post
title: "Disruptor with MultiProducer"
date: 2014-02-04 10:00
author: scooletz
permalink: /2014/02/04/disruptor-with-multiproducer/
nocomments: true
categories: ["Architecture"]
tags: ["disruptor", "multiproducer", "performance", "threading"]
imported: true
---

I hope you're aware of the LMAX tool for fast in memory processing called [disruptor](http://lmax-exchange.github.io/disruptor/). If not, it's a must-see for nowadays architects. It's nice to see your process eating messages with speeds ~10 millions/s.
One of the problems addressed in the latest release was a fast multi producer allowing one to instantiate multiple tasks publishing data for their consumers. I must admit that the simplicity of this robust part is astonishing. How one could handle claiming and publishing a one or a few items from the buffer ring? Its easy, claim it in a standard way using CAS operation to let other threads know about the claimed value and publish it. But how publish this kind of info? Here's come the beauty of this solution:

1. allocate a int array of the buffer ring length
1. when items are published calculate their positions in the ring (sequence % ring.length)
1. set the values in the helper int array with numbers of sequences (or values got from them)

This, with overhead of int array allows:

1. waiting for producer by simply checking the value in the int array, if it matches the current number of buffer iteration
1. publishing in the same order items were claimed
1. publishing with no additionals CASes

Simple, powerful and fast.
Come, take a look at it: [MultiProducerSequencer](https://github.com/LMAX-Exchange/disruptor/blob/master/src/main/java/com/lmax/disruptor/MultiProducerSequencer.java)
