---
layout: post
title: "Using Fody to provide common parts for structs"
date: 2016-04-01 09:00
author: scooletz
permalink: /2016/04/01/using-fody-to-provide-common-parts-for-structs/
nocomments: true
categories: ["C#", "RampUp"]
tags: ["emit", "Fody", "MSIL", "RampUpNet"]
imported: true
---

The [RampUp](https://github.com/Scooletz/RampUp) library is meant to provide low-latency, low/no-alloc environment for building fast systems in .NET. As it's based on messaging in an actor-like/SEDA fashion, the messages are the first class citizen in its environment. Because of these requirements, unlike in other frameworks/systems, they've been built on structs. Yes, good old fashioned value types that has no virtual method tables, no object overhead. They're just pure data. But even in the world of pure data sometimes you need a common denominator, which provides some basic information. Let me share my RampUp approach to this problem.

### Envelope

In case of RampUp and its messages, the part that should be attachable to every message is an envelope. You probably want to now the sender of the message and maybe a few more facts. We can't derive as structure types cannot derive one from another. How can this be done, how to introduce at least one common field in all the messages? Having one field of type Envelope would be sufficient as we could use this field to store all the needed information.

### Fody

There's a tool created by Simon Cropp called [Fody](https://github.com/Fody/Fody). It's a AOP tool, a weaver, a post compiler. With this you can create *ModuleWeavers*, that are reusable (there's a lot of them) and/or applied only in the solution they were created. Using this tool I've been able to deliver a [weaver that scans](https://github.com/Scooletz/RampUp/blob/master/src/RampUp.Enveloper.Fody/ModuleWeaver.cs) for messages in a project and adds a specific envelope field. For each message a metadata is created describing the offset to the *Envelope* field. Additionally, on the basis of the metadata, a message reader and a writer are emitted so that the final user of RampUp does not need to access this field manually.

### Summary

Using a post compiler is often seen as an overkill. On the other hand, being able to introduce a common denominator for a set of value types is impossible without either manual copy-paste techniques or weaving it in a post compilation process. I prefer the latter.
