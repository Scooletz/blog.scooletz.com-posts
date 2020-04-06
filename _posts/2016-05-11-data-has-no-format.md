---
layout: post
title: "Data has no format"
date: 2016-05-11 09:00
author: scooletz
permalink: /2016/05/11/data-has-no-format/
categories: ["Architecture", "Design", "Uncategorized"]
tags: ["architecture", "data", "design"]
imported: true
---

*I need to be able to store 1GB of JSON*

*I'd like to push XML 100 MB/s to this Azure blob*

*I need to log this data as CSV*

Statements like this are sometimes true, but in the majority of cases the format is not given and is a part of designing your architecture/application. Or redesigning if needed. Selecting a proper format can lower the size of your data, increasing the throughput of your system, if a medium like a disk or a network is saturated. That's why systems like [Apache Arrow](https://github.com/apache/arrow/blob/master/format/Layout.md) or [Google's Dremel ](http://research.google.com/pubs/pub36632.html)use their own formats. That's why you may consider using the [protobuf-net serialization ](https://github.com/mgravell/protobuf-net)for EventStore, disabling it build in v8 projections and lowering size of events at the same time. For low latency systems you can choose the new library [Simple Binary Encoding](https://github.com/real-logic/simple-binary-encoding). That's why sometimes storing data in another format is simply better. I've written a blog post [Do we really need all these data tranformations](http://blog.scooletz.com/2015/01/14/do-we-really-need-all-these-data-transformations/) and it doesn't state something opposite. It's all about making a rational and proper choices of the storage format and taking into consideration different aspects of it and its influence on your system. With this one decision you might improve your system performance greatly.
