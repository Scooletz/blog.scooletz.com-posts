---
layout: post
title: "Event Driven Architecture - feed your head"
date: 2015-03-24 12:43
author: scooletz
permalink: /2015/03/24/event-driven-architecture-feed-your-head/
categories: ["DDD", "Event sourcing"]
tags: ["CQRS", "EDA", "EventSourcing"]
imported: true
---

It's been a few days since the last [Warsaw .NET User Group meeting](http://www.meetup.com/WG-NET/events/220966958/ "Last WG-NET meeting"). The main presentation was provided by me & Tomasz Frydrychewicz. The title was: "Event Driven Architecture in practice". Being given a high number of answers to the pool and the overall was very positive response I may call it one of my best presentations ever. Anyway, I was being asked many questions during these days, the main one is what/who should I read/watch to immerse into this event-based approach. The list below tries to answer it somehow, grouped by author:

1. Martin Fowler

    1.  [http://martinfowler.com/eaaDev/EventSourcing.html](http://martinfowler.com/eaaDev/EventSourcing.html) - the top 1 Google search result. Martin provides a good intro, mixing a bit a concept of storying commands and events. Anyway, this is a must read if you starts with this topic

    2.  [http://martinfowler.com/eaaDev/RetroactiveEvent.html](http://martinfowler.com/eaaDev/RetroactiveEvent.html) - the article which one should become familiar with after spending some time with event modelling. Some domains are less prone to result in special cases for handling this kind of events, other may be very fragile and one should start with this
1. Lokad, CQRS, Rinat Abdullin

    1.  [http://lokad.github.io/lokad-cqrs/](http://lokad.github.io/lokad-cqrs/) - a must-read if you want to choose the event way. Plenty of materials and tooling. To me some parts are a bit frameworkish, but still, it's one of the best implementations I've seen. Understanding this might be your game changer.

Additionally, it provides an Azure storage implementation.

1. Rinat Abdullin & Kerry Street

    1.  [Being the worst](http://beingtheworst.com/) - how to become a master? Immerse yourself in a new field as the worst. That's how winning is done! Am amazing journey through learning about DDD, Event Sourcing and many paradigms.
1. Microsoft Patterns and Practices:

    1.  [CQRS Journey](https://msdn.microsoft.com/en-us/library/jj554200.aspx) - a free book about a group of developers using event driven approach with DDD in mind, to build a new system. I love the personas they use to drive dialogues between different opinions/minds/approaches. It's not a guide. I'd rather consider it a diary of all the different cases you can meet when implementing solutions using these approaches.
1. Event Store

    1.  The whole [Event Store database](https://github.com/EventStore) is an actual event store for storying events from the event sourced systems. I encourage you to spend a week or more on reading its code. It's a good codebase.

    2.  [Event sourcing documentation](http://docs.geteventstore.com/introduction/event-sourcing-basics/) is a short introduction to the ES world. After all these years, it still uses the Word generated pictures :) but this doesn't diminish its value.
1. NEventStore

    1.  [NEventStore](https://github.com/NEventStore) is an open source library for storying and querying your events. It's opinionated, for instance it stores all the events as one commit object. I've read it carefully, although I don't like its approach still. One should read it though, it's always worth to know what's already provided.

It's a bit long list but nobody said that you can learn a new paradigm over one weekend. So read, learn and apply it successfully :)
