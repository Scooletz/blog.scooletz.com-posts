---
layout: post
title: "RampUp for systems with the mechanical sympathy"
date: 2016-03-01 11:00
author: scooletz
permalink: /2016/03/01/rampup-for-systems-with-the-mechanical-sympathy/
categories: ["RampUp"]
tags: ["RampUpNet"]
imported: true
---

[https://github.com/Scooletz/RampUp](https://github.com/Scooletz/RampUp)

That's the very first post about RampUp, a project I've been working on for a while. RampUp is a .NET library providing a performant low/no-alloc environment for demanding systems. It's based on understanding the modern hardware and applying the [mechanical sympathy](https://groups.google.com/forum/#!forum/mechanical-sympathy). By understanding the modern hardware, I mean CPU mostly, but as RampUp is about the journey, not the path, some other may be involved as well (I don't mean the user mode networking I hope).

### The journey, not the path

RampUp has been started as a journey project and it's still in this phase. The goal of this journey is simple: provide a high level abstraction, a layer that enables writing extremely performant systems (probably not applications) in C#/.NET for modern hardware. Initial tests show that this approach may be valid. The provided infrastructure is able to handle 1 million messages in ~100 miliseconds on a single machine. Yes, that's 10 millions per second without any allocations at all! It's worth to mention that messages are sent by two publishers and there's only one consumer! As the initial results are promissing, I'm aiming at ending this journey with a real OSS product for building highly demanding systems.

### The inspirations

There are many great projects out there that are an inspiration for RampUp:

* [EventStore](https://github.com/EventStore/EventStore) - an event database; it uses SEDA approach and an in-memory messaging
* [Akka.NET](https://github.com/akkadotnet/akka.net) - a very active actor's framework for .NET
* [Aeron](https://github.com/real-logic/Aeron) - an extremely performant messaging system using UDP + negative acknowledgements; brought to live by Martin Thomson, the Java mechanical sympathy guru

### I want to know more

If you're interested in the project and the approach described above, I'll be speaking about RampUp in local group meetings in Poland. Hopefully, we'll have a chance to discuss it for a while.

### #dajsiepoznac

There's an ongoing contest for the Polish development community, being lead by Maciej Aniserowicz ([here](http://www.maciejaniserowicz.com/daj-sie-poznac/)). It was a positive kick that has lead me to publishing RampUp right now. There's a lot of work to be done but as the intial tests were so positively shocking, why not to share it right now?
