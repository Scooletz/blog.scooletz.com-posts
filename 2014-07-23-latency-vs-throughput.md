---
layout: post
title: "Latency vs throughput"
date: 2014-07-23 09:00
author: scooletz
permalink: /2014/07/23/latency-vs-throughput/
nocomments: true
categories: ["Design"]
tags: ["design", "latency", "statistics", "throughput"]
imported: true
---

There are two terms which you should consider during designing your system. The more robust, the bigger system you design the deeper should be your understanding of these two values.

### Throughput

 Throughput is nothing more than number of operations per given unit of time which can be processed by your system. For instance, in a web site case one may want to easily handle one thousand requests per second. To define needed throughput you can use estimation like

> given the number of users concurrently using system set to 1000,
>  given the estimated number of users actions per second set to 1,
>  the system should have throughput equal to 1000 req/s

Is it a good estimation? I'd reconsider for sure:

1. peak values of concurrent users. In majority of systems there are hours where your servers do nothing. On the other hand, there are hours where all of your users are logged in
1. number of actions per second. The value 1 operation/s may be good for a person seeing a computer for the very first time. It's much lower than standard PC user response

The obvious operation one can do to increase the throughput is batching. It's easier to write and fsync/FlushFileBuffers after writing a batch of entries rather than syncing all the time. The same goes with network IO. Sending a bigger frame containing more messages would lead to increased throughput.

### Latency

 Latency is a time till request completion. You should forget about silly average value and go for [median](http://en.wikipedia.org/wiki/Median), [quartile ](http://en.wikipedia.org/wiki/Quartile)and [percentile](http://en.wikipedia.org/wiki/Percentile), especially 99%, 99.9% and more. Don't be fooled by calculating average latency across whole day. Especially for systems with lots of load, these many nines will be more common than you think. To get a taste of it you should watch definitely [Gil Tene discussing some common pitfalls encountered in measuring and characterizing latency](http://www.infoq.com/presentations/latency-pitfalls).

### Throughput vs latency

 Having this definition, is it good enough to ask for maximized throughput? My answer is that it isn't.

> Without defined and measured latency, throughput can be bounded by the most optimal batching requests for the slowest resources.

You should satisfy other requirements as well, or at least provide meaningful statistics like [MBeans of Cassandra DB ](http://www.datastax.com/documentation/cassandra/1.2/cassandra/operations/ops_monitoring_c.html) or [EvenStore queues lengths](https://github.com/EventStore/EventStore/wiki/Architectural-Overview).
