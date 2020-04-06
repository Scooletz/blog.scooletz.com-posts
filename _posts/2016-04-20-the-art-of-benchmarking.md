---
layout: post
title: "The art of benchmarking"
date: 2016-04-20 10:00
author: scooletz
permalink: /2016/04/20/the-art-of-benchmarking/
categories: ["Profiling", "RampUp", "Testing"]
tags: ["benchmarking", "profiling", "RampUpNet"]
imported: true
---

I've been told that [Akka ](http://akka.io/)can process 50 millions of messages per second on a laptop. This isn't the number you hear every day, even if you write performance focus applications.

I've been recently optimizing my [RampUp](https://github.com/Scooletz/RampUp) library and I know that it can perform well, but reaching 50 millions of messages on my 4 hardware cores? That would be a hard thing to do. Possible, maybe, if the test was designed in a way that it groups cores in some way... The current official number is 10 millions msg/s on my laptop and the test uses two producers trying to flood a single consumer. It's a multi producer single consumer scenario. But let's go back to the Akka benchmark.

The best performance marked with 'single machine' phrase is [this. ](http://letitcrash.com/post/20397701710/50-million-messages-per-second-on-a-single)It actually was able to process 48 millions of messages on a single machine! That's great. Let's take a look what kind of machine is that

* **Processor: 48 core AMD Opteron (4 dual-socket with 6 core AMD® Opteron™ 6172 2.1 GHz Processors)**
* **Memory: 128 GB ECC DDR3 1333 MHz memory, 16 DIMMs**
* OS: Ubuntu 11.10
* JVM: OpenJDK 7, version “1.7.0_147-icedtea”, (IcedTea7 2.0) (7~b147-2.0-0ubuntu0.11.10.1)
* JVM settings: -server -XX:+UseNUMA -XX:+UseCondCardMark -XX:-UseBiasedLocking -Xms1024M -Xmx2048M -Xss1M -XX:MaxPermSize=128m -XX:+UseParallelGC
* Akka version: 2.0
* **Dispatcher configuration other than default:**

    **parallelism 48 of fork-join-exector**
    **throughput as described**

It's not a laptop. It's not a usual single machine. It's a quite powerful server with a special dispatcher used to get this performance.

I'm not saying, that it's bad to use good hardware for your tests. I'm not trying to defend RampUp performance, as it does not compete with Akka - it's for different purposes. I'm just saying that providing benchmarks, shouldn't be focused on providing number only. There is so much more information needed to give the whole background for a test. Again, the way of communicating number depend if one wants to sell sth providing numbers or provide real results. If you want the second, choose your words wisely.
