---
layout: post
title: "Shallow and deep foundations of your architecture"
date: 2017-03-27 09:55
author: scooletz
permalink: /2017/03/27/shallow-and-deep-foundations-of-your-architecture/
nocomments: true
image: /img/2017/03/stocksnap_b84bpongiv.jpg
categories: ["Architecture", "Design"]
tags: ["architecture", "design"]
imported: true
---

### TL;DR

This entry addresses some of the ideas related to various types of foundations one can use to create a powerful architectures. I see this post as somewhat resonating with the Gregor Hohpe approach for [architecture selling options](https://www.linkedin.com/pulse/architecture-selling-options-gregor-hohpe).

### Deep foundations

The deep/shallow foundations allegory came to me after running my workshop about event sourcing in .NET. One of the main properties of an event store was the fact whether it was able to provide a linearized view of all of its events. Having or not this property was vital for providing a simple marker for projections. After all, if all the events have a position, one could easily track only this one number to ensure, that an event was processed or not.

This property laid out a strong foundation for simplicity and process management. Having it or not, was vital for providing one design or another. This was a deep foundation, that was doing a lot of heavy-lifting of the design of processes and views. Opting out later on, from a store that provides this property, wouldn't be that easy.

You could rephrase it as having strong requirements for a component. On the other hand, I like to think about it as a component providing deep foundations for the emerging design of a system.

### Shallow foundations

The other part of a solution was based on something I called *Dummy Database*. A store that has only two operations PUT & GET, without transactions, optimistic versioning etc. With a good design of the process that can store its progress just by storing a single marker, one could easily serialize it with a partition of a view and store it in a database. What kind of database would it be? Probably any. Any SQL database, Cassandra or Azure Storage Tables are sufficient enough to make it happen.

### Moving up and down

With these two types of foundations you have some potential for naturally moving your structure. The deep foundations provide a lot of constraints that can't be changed that easily, but the rest, founded on the shallow ones, can be changed easily. Potentially, one could swap a whole component or adjust it to an already existing infrastructure. It's no longer a list of requirement to satisfy for a brand new shiny architecture of your system. The list is much shorter and it's ended with a question "the rest? we'll take whatever you have already installed".

### Summary

When designing, split the parts that need to be rock solid and can't be changed easily from the ones that can. Keep it in mind when moving forward and do not poor to much concrete under parts where a regular stone would just work.
