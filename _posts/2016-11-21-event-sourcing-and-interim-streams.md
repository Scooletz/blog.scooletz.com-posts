---
layout: post
title: "Event Sourcing and interim streams"
date: 2016-11-21 09:55
author: scooletz
permalink: /2016/11/21/event-sourcing-and-interim-streams/
nocomments: true
image: /img/2016/11/stocksnap_fadefe2a4d.jpg
categories: ["DDD", "Event sourcing"]
tags: ["event driven architecture", "EventSourcing"]
imported: true
---

### TL;DR

When modelling with event sourcing, people often tend to create long living streams/aggregates. I encourage you to improve your modelling with interim streams.

### Long live the king

A user, an account, a company. Frequently this kinds of aggregates are distinguished during first modelling attempts. They are long lasting, never ending streams of events. Sometimes they combine events from different contexts, which is a smell on its own. The extended longevity can be a problem though and even when events are correlated and an aggregate is dense, in the business meaning, rethinking it's lifetime can help you design a better system.

### Interim streams

One of the frequent questions raised when applying event sourcing is: *how do I ensure that a user is registered with a unique email*? For sure in majority of cases handling duplicates by a person would be just enough (how many people would actually try to register twice...), but let's try to make this a real business requirement.

Because of the disability to span a transaction across different streams, we can model this requirement in a different way. Let's start with creation of a stream which name is derived from email, like:

*useraggregate_email@domain.com*

This would ensure, that for a specific email, only one aggregate can be created. Next step is to put an event holding all the needed data to add a user, like:

UserRegistered:

{name: "Test", surname: "Test", phone: "111111111111", userid: "some_guid"}

The final step is to have a projection that consumes this event and appends it to a new stream named:

*useraggregate_some_guid*

This ensures that we don't use a natural key and still preserve the uniqueness requirement. At the end we have an aggregate that can be identified with its GUID.

### Time to die

The interim stream example with the unique email wasn't the best one, but was good enough that a stream can make interim streams or that can be based on an interim stream. It's vital to liberate from thinking that every stream needs to live forever. I'd say that in majority of cases there's a moment when a stream needs to ends its life.

![stocksnap_6qj9xjwicj](/img/2016/11/stocksnap_6qj9xjwicj.jpg)

The longevity isn't the most important virtue of a stream. It's created either to model a behavior in a right way or be just a passage for other streams. Next time when you model a stream ask yourself, how long will it live?
