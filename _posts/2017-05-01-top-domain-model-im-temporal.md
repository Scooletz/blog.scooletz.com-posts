---
layout: post
title: "Top Domain Model: I'm temporal"
date: 2017-05-01 08:55
author: scooletz
permalink: /2017/05/01/top-domain-model-im-temporal/
nocomments: true
image: /img/2017/04/top_domain_model.jpg
categories: ["DDD", "Event sourcing"]
tags: ["domain driven design", "EventSourcing", "Top Domain Model"]
imported: true
---

### TL;DR

This is the first from entries written under wings of [Top Domain Model](http://blog.scooletz.com/2017/04/24/top-domain-model). Nope, it's not a new TV show for domain driven modellers. It's a series about domain driven modelling itself. Today, we'll take a look at models and their temporal dimension.

### Reservation

Imagine that you model a system for a major hospital. Unfortunately, the registrations are opened once a year, and multiple people want to register at the same time. It's even worse, they want to register to get their procedure scheduled in a specific room that has all the needed tools. The first idea could be to model the room as an aggregate. You can imagine hundreds of people trying to update the same aggregate. It won't work.

### Temporal dimension

To address this you could introduce a temporal dimension of the room. Instead of treating it as one room, you could model it as a *RoomDay*, that represents a schedule for a specific day. This would:

* spread the saturation over multiple aggregates (possibly with a more less natural distribution)
* make the aggregate time-bound (once the day passes, whole aggregate can be archived, deleted)

You could chop even further if needed, nevertheless, adding a temporal dimension to an aggregate can easily help you to simplify challenges related to scale and long lifespan of aggregates.

### Summary

When modelling an aggregate, that it's going to be highly contended, ask yourself is it naturally temporal? Possibly it can be replaced by multiple aggregates, each assigned to one year/month/day.
