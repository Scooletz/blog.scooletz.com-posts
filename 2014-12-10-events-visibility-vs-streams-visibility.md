---
layout: post
title: "Events visibility vs streams visibility"
date: 2014-12-10 11:00
author: scooletz
permalink: /2014/12/10/events-visibility-vs-streams-visibility/
nocomments: true
categories: ["Event sourcing"]
tags: ["event sourcing", "process manager", "projections"]
imported: true
---

In my recent implementation of a simple event sourcing library I had to make a small design choice. There are streams which should be considered private. For instance, there's a process manager position stream, which holds the position change of events already processed by process managers. Its' events should not be published to other modules, hence, it'd be nice to have an ability to hide them from others. The choice was between introducing some internal streams vs internal events. What would you choose and why?

My choice was to introduce internal events (a simple *InternalEventAttribute* over an event type). This lets me not only to hide systems' events but also enables people using this library to hide some, potentially internal data, within the given system/module. The reader can see the gaps in the order number of events in a module stream, but nobody beside the original module can see what was in the event.

As with every tool, it should be used wisely.
