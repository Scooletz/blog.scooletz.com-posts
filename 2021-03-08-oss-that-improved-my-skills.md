---
layout: post
title: "Open Source projects that improved my skills"
date: 2021-03-08 11:00
author: scooletz
permalink: /2021/03/08/oss-that-improved-my-skills
image: /img/2021/03/oss.png
categories: ["dotnet", "csharp", "oss"]
tags: ["dotnet", "csharp", "oss"]
whitebackgroundimage: true
---

Recently I had an interesting conversation about Open Source Software. I'm far from waving any flag related to this approach. I treat OSS as a tool that can be used to achieve one's goal. It could be sharing your knowledge, helping people by providing something for free or even building something and providing a paid support for it. Bigger players may also use it as a tactic for aggressive commoditization and ruling out the competition (using permissive licencing of their tool). There's one particular thing about OSS though. You can learn a lot out of it. In this post I want to share some .NET OSS projects that I worked with, used, and potentially, contributed to. I did my best to dive deep into memories and bring some projects that made the biggest impact on my engineering.

### protobuf-net

[protobuf-net](https://github.com/protobuf-net/protobuf-net) is a .NET implementation of Google Protocol Buffers protocol. Maintained and created by [@marcgravell](https://twitter.com/marcgravell) from StackOverflow, taught me a lot. It was one of these projects that kept me passionate about digging into it and .NET itself. It showed a real case usage of extensive IL emit usage. It's also focused on speed and performance which is aligned with my perception of what software should be. Making me interested, resulted in a few experiments like [protobuf-linq](https://github.com/Scooletz/protobuf-linq) and [Protopedia](https://github.com/Scooletz/Protopedia) that still can be found in their archived form on my GitHub profile.

Nowadays, with gRPC getting some love from .NET itself it may look like an obvious choice for serialization. A few years back though, definitely it wasn't mainstream. I still remember sprinkling WCF with `protoEndpointBehavior` to make it use protobufs as a serializer...

### NHibernate

[NHibernate](https://github.com/nhibernate/nhibernate-core) was started as a .NET port of Java's Hibernate. I must admit that I spent unhealthy amounts of time reading the codebase, learning ideas behind it and abusively extending it in different projects. After all these years, it'd be my first project to mention if somebody asked me where to learn how ORM works. I'm aware that my answer is a bit outdated, after moving Entity Framework to OSS, but still, I feel a kind of sentiment when good old fashioned NH is mentioned.

### Disruptor & Aeron

[Disruptor](https://github.com/LMAX-Exchange/disruptor) and [Aeron](https://github.com/real-logic/aeron) are really close in memories. The reason for this might be the person of [Martin Thompson](https://twitter.com/mjpt777) who co-created both of them. They address particular needs of processing high volumes of events really fast. To make that happen, they use really low-level concurrency primitives. If you think about finding a project where `volatile`, memory barriers and memory-mapped files are bread and butter, this is it.

Disruptor focuses on providing a queue-like structure, precisely a ring buffer, that can be used by event processors composed in a DAG. Think of it as computations that are dependent on each other but have no cycles in the gragh (hence DAG). Aeron is a different beast though. It brings messaging to another level. With its latest adoption of [Raft consensus algorithm](https://raft.github.io) for the Aeron Cluster, it introduces a whole bunch of aspects related to distributed databases. I must admit, that it's a great learning source for both, low-level performance as well as application of the consensus algorithms.

### Marten

[Marten](https://github.com/JasperFx/marten) is a really interesting .NET library that uses native PostgreSQL capabilities. On top of that it delivers both, a document database and an event store, that you can use for event sourcing, with the support for projections and view generation. My small involvement in this project was related purely to the performance/memory management aspect of it (see [my PRs](https://github.com/JasperFx/marten/pulls?q=is%3Apr+author%3AScooletz+is%3Amerged)), but I did my work studying the codebase and the docs. At the same time it was interesting, because it required to provide a buffer related capability in [npgsql](https://github.com/npgsql/npgsql) driver to make it accept array segments. Beside the coding aspect of it, I really enjoyed all the small interactions that we had. I think this might be the first time when I had a conversation with [Shay](https://twitter.com/shayrojansky).

### BenchmarkDotNet

[BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) is an awesome microbenchmarking tool that shows you statistical values of benchmarked code. It also makes the results comparable, so that you can compare outputs from different branches and see what's different. Having ability to compare makes all your learning much better, because you can run experiments which are vital for any kind of learning.

The overall impact of this project on modern .NET development cannot be overstated. My perception is that, if you issue a performance related PR and you don't provide a BDN report, you may be missing something.

It's worth to notice that BenchmarkDotNet was moved to [.NET Foundation](https://dotnetfoundation.org) and now is under its wings.

### NServiceBus

[NServiceBus](https://github.com/Particular/NServiceBus) is product that is build Particular Software, a company led by [Udi Dahan](https://twitter.com/UdiDahan). I had a pleasure to work in there for over 3 years. As we're talking about OSS experience and impact, I want to mention a particular (a friendly pun intended) thing about the product that Particular provides. And I don't mean the library nor the powerful platform that is provided.

What I find truly extraordinary is the way the documentation is treated. If you take a look at [docs](https://docs.particular.net/nservicebus/) you'll see a vast number of topics. If you follow one of the links, you may find a version header that allows you to switch the look of this specific page to a different version of the component! Being involved in a product where features must be properly documented (when I reread this sentence it sounds terribly corporate-ish) because the documentation is a part of a product!

### Nethermind

[Nethermind](https://nethermind.io) is a .NET client for Ethereum that I somewhat was covered in my entry about [improving Nethermind performance](/2020/11/23/improving-Nethermind-performance). I want to mention it though, as it provides a really interesting case for learning about real-life performance. Ethereum provides a unique capability of running programs (smart contracts), which requires a virtual machine to get the code executed. Additionally, it requires a lot of work with hashing and other kinds of computation, making this project probably TOP 1 when thinking about .NET projects driven by computation. I know, this kind of projects isn't that popular. At the same time it makes it even more interesting.

### .NET

[.NET runtime](https://github.com/dotnet/runtime) itself is open. And this is the biggest game changer for .NET folks I believe. Being able not only to read code but also get involved in discussions and sneak peak how the development process works. Who wouldn't like that?! The amount of work that is being done in this repository shows how complex and big the project is. The right application of labels makes it easier to navigate though. If you're interested in a specific area, just search for a specific `area-*` label and you'll find all the related issues.

### Summary

I hope that you'll visit some of these projects and that they'll provide some food for thought. If you don't have your own list of "favorite" OSS projects I encourage you to spend some time and make one. It's a really refreshing feeling to see all the shoulders of giants that modern engineering stands on.
