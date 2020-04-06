---
layout: post
title: "Event stores and event sourcing: some not so practical disadvantages and problems"
date: 2017-03-20 09:55
author: scooletz
permalink: /2017/03/20/event-stores-and-event-sourcing-some-not-so-practical-disadvantages-and-problems/
image: /img/2017/03/stocksnap_oxbtpheyth.jpg
categories: ["DDD", "Design patterns", "Event sourcing"]
tags: ["EventSourcing"]
imported: true
---

### TL;DR

This post is some kind of answer to [the article](http://www.ben-morris.com/event-stores-and-event-sourcing-some-practical-disadvantages-and-problems/) mentioned in a tweet by Greg Young. The blog post of the author has no comment section. Also, this post contains a lot of information, so that's why I'm posting it instead of sending as an email or DM.

### Commits

> Typically, an event store models commits rather than the underlying event data.

I don't know what is a typical event store. I know though that:

1. [EventStore](https://geteventstore.com/) built by Greg Young company, a standalone event database, fully separates these two, providing granularity on the event level
1. [StreamStone](https://github.com/yevhen/Streamstone) that provides support for Azure Table Storage, works on the event level as well
1. [Marten](http://jasperfx.github.io/marten/) , a PostgreSQL based document&event database also works on the singular event level

For my statistical sample, the quoted statement does not hold true.

### Scaling with snapshots

> One problem with event sourcing is handling entities with long and complex lifespans.

and later

> Event store implementations typically address this by creating [*snapshots*](http://blog.jonathanoliver.com/event-sourcing-and-snapshots/) that summarize state up to a particular point in time.

and later

> The question here is when and how should snapshots be created? This is not straightforward as it typically requires an asynchronous process to creates snapshots in advance of any expected query load. In the real world this can be difficult to predict.

The first and foremost, if you have aggregates with long and complex lifespans, it's your responsibility because you chose a model where you have aggregates like that. Remember, that there **are no right or wrong models, only useful or crappy ones**.

The second. Let me provide an algorithm for snapshoting. If you retrieved 1000 events to build up aggregate, you should snapshot it (serialize + put into cache in memory + possibly store in a db). Easy and simple, I see no need for fancy algorithms.

### Visibility of data

> In a generic event store payloads tend to be stored as agnostic payloads in JSON or some other agnostic format. This can obscure data and make it difficult to diagnose data-related issues.

If you as an architect or developer know your domain and you know that you need a strong schema, because you want to use it as [published interface](https://www.martinfowler.com/bliki/PublishedInterface.html) but still persist data in JSON instead of some schema-aware serialization like protobuf (binary, schema-aware serialization from Google) it's not the event store fault. Additionally,

1. EventStore
1. StreamStone

both handle binary just right (yes, you can't write js projections for EventStore, but still you can subscribe).

### Handling schema change

> If you want to preserve the immutability of events, you will be forced to maintain processing logic that can handle every version of the event schema. Over time this can give rise to some extremely complicated programming logic.

It was shown, that instead of cluttering your model with different versions (which still, sometimes it's easier to achieve), one could provide a mapping that is applied on the event stream before returning events to the model. In this case, you can handle the versioning in one place and move forward with schema changes (again, if it's not your published interface). This is not always the case, but this patter can be used to reduce the clutter.

### Dealing with complex, real world domains

> Given how quickly processing complexity can escalate once you are handling millions of streams, it’s easy to wonder whether *any* domain is really suitable for an event store.

EventStore, StreamStone - they are designed to handle these millions.

### The problem of explanation fatigue

> Event stores are an abstract idea that some people really struggle with. They come with a high level of “explanation tax” that has to be paid every time somebody new joins a project.

You could tell this about messaging and delivery guarantees, fast serializers like protobuf or dependency injection. Is there a project, when a newbie joins, they just know what and how to do it? Nope.

### Summary

It's your decision whether to use event sourcing or not, as it's not a silver bullet. Nothing is. I wanted to clarify some of the misunderstandings that I found in the article. Hopefully, this will help my readers in choosing their tooling (and opinions) wisely.
