---
layout: post
title: "Events on the Outside versus Events on the Inside"
date: 2016-05-23 09:00
author: scooletz
permalink: /2016/05/23/events-on-the-outside-versus-events-on-the-inside/
categories: ["Architecture", "DDD", "Event sourcing"]
tags: ["event driven architecture", "event sourcing"]
imported: true
---

Recently I've been revisiting some of my Domain Driven Design, CQRS & Event Sourcing knowledge and techniques. I've supported creation of systems with these approaches, hence, I could revisit the experiences I had as well. If you are not familiar with these topics, a good started could be my [Feed Your Head](https://blog.scooletz.com/2015/03/24/event-driven-architecture-feed-your-head/) list.

### Inside

So you model you domain with aggregates in minds, distilling contexts and domains. The separation between services may be clear or a bit blurry, but it looks ok and, more important, maps the business well. Inside a single context bubble, you can use your aggregates' events to create views and use the views when in need of data for a command execution. It doesn't matter which database you use for storing events. It's simple. Restore the state of an aggregate, gather some data from views, execute a command. If any events are emitted, just store them. A background worker will pick them up to dispatch to a [Process Manager](https://blog.scooletz.com/2014/11/21/process-manager-in-event-sourcing/).

### Outside

What about exposing you events to other modules? If and how can another module react to an event? Should it be able to build it's own view from the data held in the event? All of these could be sum up in one question: do external events match the internal of a specific module? My answer would be: it's not easy to tell.

In some systems, these may be good. By the system I mean not only a product, but also a team. Sometimes having a feed of events can be liberating and enabling faster grow, by speeding up initial shaping. You could agree to actually separate services from the very start and verify during a design, if the logical complexity is still low. I.e., if there is not that much events shared between services and what they contain.

This approach brings some problems as well. All the events are becoming your API. They are public, so now they should be taken into consideration when versioning your schemas. Probably some migration guide will be needed as well. The bigger public API the bigger friction with maintaining it for its consumers.

Having this said, you could consider having a smaller and totally separate set of events you want to share with external systems. This draws a visible line between the Inside & the Outside of your service, enabling you to evolve rapidly in the Inside. Maintaining a stable API is much easier then and the system itself has a separation. This addresses questions about views as well. Where should they be stored originally. The answer would be to store properly versioned, immutable views Inside the service, using identifiers to pass the reference to another service. When needed, the consumer can copy & transform the data locally. A separate set of events provides ability to do not use Event Sourcing where not needed. That kind of options (you may, but don't have to use it) are always good.

For some time I was an advocate of sharing events between services easily, but now, I'd say: apply a proper choice for your scenario. Consider pros and cons, especially in terms of the schema maintainer tax & an option for not sticking to Event Sourcing.

### Inspirations

The process of revisiting my assumptions has been started by a few materials. One of them is a presentation by Andreas Ohlund, ['Putting your events on a diet'](https://skillsmatter.com/skillscasts/2990-events-diet), sharing a story about deconstructing an online shop into services. The second are some bits from [ A Decade of DDD, CQRS, Event Sourcing by Greg Young](https://www.youtube.com/watch?v=LDW0QWie21s). The last but not least, Pat Helland's [Data on the Outside versus Data on the Inside](http://cidrdb.org/cidr2005/papers/P12.pdf).
