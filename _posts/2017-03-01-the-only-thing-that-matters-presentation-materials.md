---
layout: post
title: "The Only Thing That Matters - presentation materials"
date: 2017-03-01 09:55
author: scooletz
permalink: /2017/03/01/the-only-thing-that-matters-presentation-materials/
image: /img/2017/02/only.png
categories: ["presentations"]
tags: []
imported: true
---

### TL;DR

A few days ago I gave a presentation titled *The Only Thing That Matters*. Below you can find the list of materials one could use to ramp up his knowledge about topics covered in the presentation. Beware! Spoilers ahead!

### Sql Server

* [http://rusanu.com/blog/](http://rusanu.com/blog/) - the first and the best source that you can find when reading about SQL Server underlying storage capabilities is this blog, especially [http://rusanu.com/2012/01/17/what-is-an-lsn-log-sequence-number/](http://rusanu.com/2012/01/17/what-is-an-lsn-log-sequence-number/)
* [https://msdn.microsoft.com/en-us/library/ms190411.aspx](https://msdn.microsoft.com/en-us/library/ms190411.aspx)
* [https://msdn.microsoft.com/en-us/library/ms179355.aspx](https://msdn.microsoft.com/en-us/library/ms179355.aspx)

### Event Store

* [https://github.com/EventStore/EventStore](https://github.com/EventStore/EventStore) - the most important part is the codebase located on GitHub
* [https://github.com/EventStore/EventStore/wiki/Architectural-Overview](https://github.com/EventStore/EventStore/wiki/Architectural-Overview) - you should take a look at the architecture overview as well. The wiki on GitHub is not maintained but still, it's valid.
* [http://blog.scooletz.com/2014/06/11/pearls-eventstore-transaction-log/](http://blog.scooletz.com/2014/06/11/pearls-eventstore-transaction-log/) - the last but not least is my entry where I describe the design behind efficient flushing of the log

### Kafka

* [https://aphyr.com/posts/293-call-me-maybe-kafka](https://aphyr.com/posts/293-call-me-maybe-kafka) - Kafka Jepsen test
* [https://kafka.apache.org/documentation/#design](https://kafka.apache.org/documentation/#design) - the design behind this broker

### Summary

I hope this links help you in pursuing the understanding of logging and append only structures. Let's flush some data on disk!
