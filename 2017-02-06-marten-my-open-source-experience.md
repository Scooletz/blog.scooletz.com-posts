---
layout: post
title: "Marten: my Open Source experience"
date: 2017-02-06 09:55
author: scooletz
permalink: /2017/02/06/marten-my-open-source-experience/
image: /img/2017/01/banner.png
categories: ["Databases"]
tags: ["database", "Marten", "Open Source"]
imported: true
---

### TL;DR

This post describes some of my Open Source experiences when working on [Marten](https://jasperfx.github.io/marten/), a *Polyglot Persistence for .Net Systems using the Rock Solid Postgresql Database (TM).* This isn't me boasting, but rather sharing a pleasant story about getting involved in some solid OSS.

### Background

The main person responsible for Marten and the main committer is [Jeremy D. Miller](https://jeremydmiller.com/). He's a well known person in .NET Open Source world as the author of StructureMap. I had a pleasure working with him before, in OSS area, when we were trying to create a better NuGet client, called [Ripple](https://github.com/FubuMvcArchive/ripple). Before creating Marten, he spent some time working with RavenDB. He described it thoroughly in [Would I use RavenDB again?](https://jeremydmiller.com/2013/05/13/would-i-use-ravendb-again/) and [Moving from RavenDB to Marten](https://jeremydmiller.com/2016/08/18/moving-from-ravendb-to-marten/). These entries are pure gold, I encourage you to read them.

### What is Marten

Marten is a client library for .NET that enables you to use Postgres database is two ways:

1. as a document database
1. as a event sourcing database

Both are included in the same library, as they share a lot. The very foundation of Marten is Postgres' ability to process JSON and treating it as the first class citizen. First class, you say, what does it even mean? You can use db parameters with JSONB type, indexes (that are truly **ACID**) and a lot more. Being given that Postgres is a mature database with a well performing storage engine, you can build a lot on top of it.

### Performance matters

I started to work on Marten after seeing the JSON serializer seam. It contained only method for serializing objects, that was returning a string

![serializer.png](/img/2017/02/serializer.png)

This means, that for every operation that involved obtaining a serialized version of an object (both: a document and an event), a new string would be allocated. For low traffic this could be good, but when operating on many documents or appending a lot of events, it isn't.

We had a short discussion in [this issue](https://github.com/JasperFx/marten/issues/640) which was followed with a [PR of mine](https://github.com/JasperFx/marten/pull/642) where I introduced a buffer pool, that provides TextWriters and uses it for upserting documents in the database. I chose to work with the *TextWriter* abstraction as all good .NET JSON serializers are capable to work with it. I encourage you to follow the PR and the issue, again it shows how a bit excessively described issue can help in communication and making things happen.

### Performance matters even more

I wanted to introduce the same pooling for appending events and bulk inserts. This was blocked by the [npgsql](http://www.npgsql.org/) Postgres .NET driver though. Because of its internal structure, when using array parameters, this driver could not extract the size of a passed text from the parameter. It wasn't supporting arrays of *ArraySegment<char>* as parameters as well... I issued the [npgsql related PR](https://github.com/npgsql/npgsql/pull/1411) to fix it and pinged [Shay](https://github.com/roji), how fast can we get it. It was released under a week.

Meanwhile, I provided PRs for [events ](https://github.com/JasperFx/marten/pull/662)and [bulk inserts](https://github.com/JasperFx/marten/pull/663) that were RED on TeamCity. Once the new client library **npgsql 3.2** has been released, Jeremy merged these two PRs making Marten allocating a lot less on the write side.

### No company, lots of work done

Jeremy, Shay and me - we share no manager, no company and we work in different time zones. Still, as a group, we were able to deliver a meaningful performance improvements for a library that we care for. This shows a true power of Open Source.

### Summary

I hope you enjoyed this possitive journey of mine. This summary does not mean I finished working on Marten. I'd say that it's totally opposite.
