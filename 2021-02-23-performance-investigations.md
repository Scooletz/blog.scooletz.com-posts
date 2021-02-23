---
layout: post
title: "Performance investigations"
date: 2021-02-23 11:00
author: scooletz
permalink: /2021/02/23/performance-investigations
image: /img/2021/02/detective.png
categories: ["performance", "programming", "dotnet"]
tags: ["performance", "programming", "dotnet"]
whitebackgroundimage: true
---

Imagine a crime scene that contains a lot of clues but no dead body. Actually, the body may be alive but not willing to answer your questions unless you're really specific. Imagine, that after solving one case, there's immediately another one! The second is a bit smaller, it follows the _diminishing returns_ law, but still, it's a never ending process. Unless you say that _this is enough_. Welcome to the performance investigation!

### It ain't dead yet

The bad performance quite often doesn't kill the ~~victim~~ app. It just makes it slower and slower to the point where it becomes noticeable by everyone. Without the big crash, the performance work is often neglected or postponed. With modern scalable architectures, money might be used to buy more servers, computation power or storage. There are a lot of remedies to address the clues without getting to the root cause. It's ain't dead yet, so maybe, there's no need to run an investigation after all.

### Gathering clues and finding the scenario

If you want to solve the case, you need to be observant and gather clues. It might be a slow process and it can take the time. Especially, if this is your first investigation! The best outcome that you can get is a repeatably bad scenario. When repeated, it should provide you with the same effects. I'm aware that you don't want to treat the app badly, but this is for the greater good.

After the profiling, searching, the performance scenario should be reproduced either with a test or a benchmark. If it requires a fancy setup, it's possible that after your 3rd or 4th attempt of solving the case, you'll get tired and stop working on it. Think about the clues and trying to make it reproducible at the same time is important as the scenario, can later be turned on into a test, a checklist or another artifact that might help in finding regressions (a.k.a. the predator strikes back).

### Got ya

The most gratifying part of the investigation is solving the case by not only answering _whodunit_ and screaming _got ya!_ but also addressing the issue by introducing a performance fix. This might be a one liner, like using a [`BigMul` method to multiply two ulong numbers](https://github.com/NethermindEth/int256/pull/15) or a huge one like [shrinking JS components of ASP.NET multiple times](https://github.com/dotnet/aspnetcore/pull/30320). Usually this is done using my tools

### Tools

There are few groups of tools that you I use. My usual setup, when working with .NET solutions includes:

- profiling with amazing JetBrains' [dotTrace](https://www.jetbrains.com/profiler) and [dotMemory](https://www.jetbrains.com/dotmemory)
- writing a meaningful benchmark with the awesome [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet)
- doing heavy-lifting with memory abstractions of `Span` & `Memory`, going `Unsafe` or using `System.Runtime.Intrinsics` to speed things up or...
- choosing different components or libraries on a much higher level (serialization etc.) or...
- questioning and redefining some aspects of an architecture

### It's done, is it

So it's done! The case is closed! We can go home, right? Almost. The investigation addressed this issue, but when working on this slightly less uses part of the system, there was another glitch. Can there be another case? Right after closing the first one? Sure! The number of cases is infinite and the new ones may be uncovered by addressing the bigger ones. Remember, that it's common to observe the _diminishing returns_ law in action, seeing less and less gains when a specific area is investigated over and over again.

### Summary

The performance investigation is based on clues that are up to you to find them and bind them in a scenario. Once you do it, and have a repeatable way of measuring performance, you can solve the cave and prove that it addressed the issue. Remember, the _diminishing returns_ are out there, so keep them in mind when working long months/years on a small area of your app.