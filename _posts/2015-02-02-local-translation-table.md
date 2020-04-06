---
layout: post
title: "Local translation table"
date: 2015-02-02 11:00
author: scooletz
permalink: /2015/02/02/local-translation-table/
nocomments: true
categories: ["Optimization"]
tags: ["contracts", "GUID", "optimization"]
imported: true
---

It's quite to common to use GUIDs as a unique identifiers across systems as they are... unique :) The GUID generation algorithm has a very important property: it's local. Your code doesn't have to contact any service or a database. It just works. There's one problem with GUIDs, they are pretty lengthy taking 16 bytes to be stored.
Consider following example: you want to provide a descriptor for a finite set of distinguishable items, like events in event sourcing. You can use:

* an event type, a string description of it to make it right. That's the idea behind event storage within EventStore.
* GUIDs and generate them them on developers machines. They will be unique but still lengthy when it comes to storing them
* assign integers, but you will need to divide integers sets between module and be very careful to do not step into another module area

There's one additional thing you can do. You can easily combine 2 and 3. Just use GUIDs on contracts, providing uniqueness, but when it comes to storing provide a translation table, persistent, ensured of its existence during start up, mapping GUIDs to ints (4 bytes) or even shorts (2 bytes). You can easily create this kind of table locally, one for each module/service, just to embrace all the contracts used by this module. This will lower the storage cost and still let you use nice properties of Guids.

Simple, easy and powerful.
