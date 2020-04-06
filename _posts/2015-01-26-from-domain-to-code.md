---
layout: post
title: "From domain to code"
date: 2015-01-26 11:00
author: scooletz
permalink: /2015/01/26/from-domain-to-code/
nocomments: true
categories: ["DDD", "Event sourcing"]
tags: ["event sourcing", "modelling"]
imported: true
---

Currently I'm helping to model one domain with Event Sourcing. A few days ago we spent ~15 minutes on modelling some cases on the whiteboard. The result of the first phase was distilling a few aggregates with events. Later on, we described some processes as we needed to react to some events. At first to much behavior was put in a process manager, to finally be moved to a separate aggregate - a process itself. The process manager was left as a simple router. Because of the strong foundations (a library) providing us at-least-once semantics with a idempotent receivers and handling process managers, the whole discussion was on a much higher level. No imperative programming, just declarative pieces of the domain itself!
A few hours later an implementation was finished. The most important thing was that **there was no mapping!**. The code was a simple representation of the model being discussed and agreed on. No ORM magic, no enterprise onion with tons of layers and mappings. It was just a simple model of this piece of a domain. And it's been working like a charm.
