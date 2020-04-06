---
layout: post
title: "Business Driven Development"
date: 2014-11-19 11:00
author: scooletz
permalink: /2014/11/19/business-driven-development/
nocomments: true
categories: ["DDD"]
tags: ["BDD", "event sourcing"]
imported: true
---

If you're into software development you've probably heard about [Behavior-driven development](http://en.wikipedia.org/wiki/Behavior-driven_development). Recently I had a discussion whether or not business people think in this way. Fortunately, I was involved in a business workshop, so I could make some observations.
The way mentioned earlier is the only language business uses to define and discuss aspects of their actions. They are some abbreviations like:
*Once we reach 1000 participants, we assign them rooms*
Which can be easily translated into

* *Given 999 participants registered*
* *When a participant registered*
* *Then the rooms are allocated*

This can be easily read by business as by developers.
If you can model your solutions towards this kind of testing, which not necessarily must be performed with tools for BDD but can be easily done by introducing [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) and structuring your tests like in [Lokad CQRS examples](https://github.com/Lokad/lokad-cqrs/blob/master/SaaS.Domain.Tests/Aggregates/User/lock_user.cs) then you can finally start to discuss business ideas with business instead of describing how your db is updated. And this, for sure makes the difference.
