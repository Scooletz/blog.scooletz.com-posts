---
layout: post
title: "Year 2016 in .NET ecosystem"
date: 2016-12-15 09:55
author: scooletz
permalink: /2016/12/15/year-2016-in-net-ecosystem/
image: /img/2016/11/stocksnap_2r1koo7h6j.jpg
categories: ["Uncategorized"]
tags: [".net", "dotnet"]
imported: true
---

### TL;DR

This was a truly amazing year for .NET ecosystem. Let's go through some of the important events in our ecosystem.

### Core & Standard

.NET Core and [Standard](https://github.com/dotnet/standard) moved closer to the mainstream development. There are applications running Core in production already. The Standard addresses the fragile point of the cross platform solutions - APIs which you can use. It had some versioning problems, but they seem to be solved for now.

### Kestrel

The ASP team with Ben A Adams are working really hard! ASP .NET Core, on Linux, is 10th in the ranking of the plain text speed. Let me say it again: Linux, .NET Core, 10th place. And it will be faster for sure.

https://twitter.com/ben_a_adams/status/798940941159571457

### Slices, memory pools and more

.NET Core and .NET at all are getting more and more attention in terms of performance. [Slices](https://github.com/dotnet/corefxlab/tree/master/src/System.Slices) (previously spans) which can be seen as a better alternative to ArraySegment abstracting access to an allocated collection of values (an array, an unsafe pointer-based memory segment). Memory pools and UTF8 strings are being brought to the mainstream. This won't be used in every project, but having these foundations is much easier to think about .NET as a platform for writing well performing systems, databases etc.

### Benchmarking

[BenchmarkingDotNet](https://github.com/dotnet/BenchmarkDotNet) isn't a library used only occasionally. Many projects embed tests based on it as a part of their tests suites. You can see the benchmarking output table in many PRs which is really good. I've heard Knuth's phrase about performance much to often. It's good to see that performance is getting attention. You don't have to benchmark everything, but as a software engineer you should be aware of the tooling and apply it when needed.

### Asyncing

Async await is globally accepted. The understanding of it seems to be better. Hopefully, we'll see less of the sync/async battles and new APIs will follow only one of these.

### Summary

It was a really good year. Let's push it even further in the next one!
