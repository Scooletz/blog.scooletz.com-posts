---
layout: post
title: "Optimisation of queries against event-sourced systems"
date: 2014-10-03 11:00
author: scooletz
permalink: /2014/10/03/optimisation-of-queries-against-event-sourced-systems/
nocomments: true
categories: ["Event sourcing"]
tags: ["EventSourcing", "http", "projections"]
imported: true
---

I hope you're familiar with [event sourcing pattern](http://martinfowler.com/eaaDev/EventSourcing.html). Switching from update-this-row to a more behavioral paradigm in designing systems, that's one of the most influential trends for sure. But how, a system storing only business deltas/events can be queried? Can it be queried at all?

### Project your views

To answer this need you can use [projections](http://goodenoughsoftware.net/2013/02/12/projections-1-the-theory/). It's nothing more than a function applied to all, or selected events (by stream id, event category, or any dimension you can get from the processed event). The similar solution one can find in [Lokad CQRS ](https://github.com/Lokad/lokad-cqrs/blob/master/SaaS.Engine/StartupProjectionRebuilder.cs)which is described in [this post](http://abdullin.com/post/getting-rid-of-cqrs-view-rebuilds/). So yes, there is a way of applying all the needed events on a view/projections, which can be queried the given parameters. Is there a way of optimizing the queries responses?

### Fast response

For sure there is! Let's take into consideration a projection replying all events for users changing their personal data, but applying only these, which modify their user name. This is probably a very infrequent operation. The projections stores id->name map, which is used for various services to display the user friendly name. What can be done to improve the service performance storing this kind of mapping?

Consider storing two event sequence numbers:

1. for remembering to which index the events were scanned
1. for remembering the last user related event which actually changed the mapping

The second can be easily used to ETag all the responses. If the operation is infrequent, the responses can be 304ed easily for long periods of time. This ETag based optimization can be applied always. The more sparse projection state changes, the better chance of reusing the client cached response.
