---
layout: post
title: "Roslyn coding conventions applied"
date: 2016-04-22 09:00
author: scooletz
permalink: /2016/04/22/roslyn-coding-conventions-applied/
nocomments: true
categories: ["Optimization", "RampUp"]
tags: ["optimization", "RampUpNet", "Roslyn"]
imported: true
---

[Roslyn](https://github.com/dotnet/roslyn) is a 'compiler as a service' provided for both VisualBasic.NET & C#. It has a thriving community of people providing new features for .NET languages. One of the most important parts of this community is a guideline how to contribute, which defines basic rules for coding and issuing pull requests. The most important part, not only from Roslyn perspective, but as general .NET guidance are [Coding Conventions](https://github.com/dotnet/roslyn/wiki/Contributing-Code#coding-conventions).

### Avoid allocations in hot paths

This is the rule, that should be close to every .NET developer heart, not only these that work on a compiler. It's not about 'premature optimization'. It's about writing performant code that actually can sustain its performance when executing its hot paths in majority of the requests. Give it a try, and when writing some code next time (today?) have this rule in mind. Awareness of this kind, may result in having no need for profiling your production system or making a dump just to know that allocating a list for every cell of two dimensional array wasn't the best approach.

### What's your hot path

That's a good question that everyone should answer on their system basis. I asked this question a few months ago for my [RampUp](https://github.com/Scooletz/RampUp/) library:

*what's the hot path for a system using message passing?*

The answer was surprisingly obvious: the message passing itself. EventStore, using a similar approach uses classes for message passing. This plus every other object creates some GC pressure. Back then, I asked myself a question, is it possible to use structs for internal process communication and come up with a good way of passing them? My reasoning was following: if I remove the GC pressure from messages, then I remove the hottest path of allocations and this can greatly improve stability of my system. Was it easy? No it wasn't as I needed to emit a lot of code and discover some interesting properties of CLR. Did it work? Yes, it did.

Next time when you write a piece of code or design a system keep *the hot path question* in your mind and answer it. It's worth it.
