---
layout: post
title: "Concurrency - ramp up your skills"
date: 2016-10-28 19:00
author: scooletz
permalink: /2016/10/28/concurrency-ramp-up-your-skills/
nocomments: true
categories: ["C#", "RampUp"]
tags: ["concurrency", "RampUpNet"]
imported: true
---

Yesterday, I gave my [Extreme Concurrency talk](http://presentations.scooletz.com/RampUp/index.html#/) at [rg-dev](http://www.meetup.com/rg-dev/events/234073133/) user group. After the talk I had some really interesting discussions and was asked to provide some resources in the low level concurrency I was talking about. So here's the list of books, talks and blog posts that can help you to ramp up your skills

Videos:

1. [C++] Herb Sutter "atomic Weapons" - it's about C++ but covers memory models in a way, that's easy to follow and learn how it works

    1.  [Part 1](https://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-1-of-2)

    2.  [Part 2](https://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Herb-Sutter-atomic-Weapons-2-of-2)

MSDN:

1. [.NET Volatile class](https://msdn.microsoft.com/en-us/library/system.threading.volatile(v=vs.110).aspx) - it has a good description of what half-barriers are and properly shows two counterparts Read & Write methods
1. [.NET Interlocked class](https://msdn.microsoft.com/en-us/library/system.threading.interlocked(v=vs.110).aspx) - the other class with a good description providing methods that are executed atomically. Basically, these methods are JITted as single assembler operations.

Code:

1. [RampUp](https://github.com/Scooletz/RampUp) - a project of mine :)
1. [JAVA] [Aeron](https://github.com/real-logic/Aeron) - the messaging library

Books:

1. [Concurrent programming on Windows](http://www.goodreads.com/book/show/2993353-concurrent-programming-on-windows) by Joe Duffy - this is a hard book to go through. It's demanding and requires a lot of effort but is the best book if you want to really understand this topic

Blogs:

1. [Volatile reads and writes](http://joeduffyblog.com/2008/06/13/volatile-reads-and-writes-and-timeliness/) by Joe Duffy
1. [Sayonara volatile](http://joeduffyblog.com/2010/12/04/sayonara-volatile/) by Joe Duffy
1. [Atomicity, volatility and immutability are different](https://blogs.msdn.microsoft.com/ericlippert/2011/06/16/atomicity-volatility-and-immutability-are-different-part-three/) by Eric Lippert - that's the last part of this series

1. [JAVA] [Psychosomatic, lobotomy, saw](http://psy-lob-saw.blogspot.com/) - the name is strange but you won't find here disturbing videos. What you'll find though, is a deep-dive into memory models.
