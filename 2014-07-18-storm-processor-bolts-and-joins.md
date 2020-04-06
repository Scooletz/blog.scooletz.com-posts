---
layout: post
title: "Storm processor - bolts and joins"
date: 2014-07-18 11:00
author: scooletz
permalink: /2014/07/18/storm-processor-bolts-and-joins/
nocomments: true
categories: ["Storm"]
tags: ["Apache", "Bolt", "Storm processor"]
imported: true
---

[Storm processor](http://storm.incubator.apache.org/ "Storm official page"), recently moved to Apache foundation is a powerful stream processing library open sourced by Twitter. It provides needed resources to scale out the processing across multiple machines, providing at-least-once guarantees or exactly-once using [Trident](https://storm.incubator.apache.org/documentation/Trident-tutorial.html). The library is based on two basic elements:

1. spouts - sources of tuple streams
1. bolts - processing units, consuming and emitting different streams of tuples

which are combined in a topology, a mesh of elements emitting and consuming events in order of data processing.
Streams, unlike [EventStore](http://geteventstore.com/) are not cheap and represent a logical flow of data rather than an aggregate boundary. One stream can be emitted by more than one spout or bolt. The further discussion of streams is beyond scope of this article.
[The bolt declarer](http://nathanmarz.github.io/storm/doc/backtype/storm/topology/BoltDeclarer.html), used in topology builder implements plenty of interfaces allowing to define consumed tuples of different streams. What it allows one to do is assigning a given bolt instance to handle a given set of tuples from a given stream of data, for example:

1. [fieldsGrouping](http://nathanmarz.github.io/storm/doc/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) lets you bind tuples from the given stream, which contain declared fields to the given instance of bolt. What it means is that tuple with a given field value will be routed to the same instance of bolt class! This provides a very powerful behavior letting you group tuples by any dimension
1. [localOrShuffleGrouping](http://nathanmarz.github.io/storm/doc/backtype/storm/topology/InputDeclarer.html#localOrShuffleGrouping%28java.lang.String%29) provides you with a great ability of routing data in the same worker process or, if the condition cannot be satisfied, move data to another worker selected 'randomly'. This, when no grouping needed, lets you improve performance by execution collocation and skipping network overhead.

The bolt isn't limited to consume tuples only from one grouping. It can join multiple streams grouped in multiple ways. This can bring another opportunity for data repartitioning. For example, application emitting streams of data like exceptions on the production environment and user transactions can easily raise an alarm when a given user experience more than one exception every 10 transactions. A simple bolt using two fields groupings can deal with it easily.
I hope this short introduction will encourage you to dive into Storm. It's a very powerful tool, especially in [Complex Event Processing](http://en.wikipedia.org/wiki/Complex_event_processing), with scale out possibilities.
