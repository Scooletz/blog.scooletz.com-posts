---
layout: post
title: "I should've used EventStore"
date: 2014-05-30 11:00
author: scooletz
permalink: /2014/05/30/i-shouldve-used-eventstore/
nocomments: true
categories: ["Event sourcing"]
tags: ["EventStore", "GetEventStore"]
imported: true
---

One of the most important features of [EventStore](http://geteventstore.com/) is an ability to ask the questions like they were asked in the past. You don't have to rerun manually all the stored information to repartition them, or to aggregate them. All you've got to do is to write a new projection which will run from the very beginning (almost always: take a loot at [scavenging](https://github.com/EventStore/EventStore/wiki/Stream-Metadata-Parameters)) till now.
There's no question you should've asked, which cannot be added later, with no mental overhead of manually rerouting data through the pipeline once again. Nice:)
