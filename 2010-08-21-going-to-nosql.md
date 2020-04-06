---
layout: post
title: "Going to NoSQL"
date: 2010-08-21 10:07
author: scooletz
permalink: /2010/08/21/going-to-nosql/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra"]
imported: true
---

Cassandra is a NoSQL database, hence there is no easy way to map your SQL experiences to this brave new world of no-relationship. The most fundamental paper, which helped me to get the idea of Cassandra was Google's white paper describing their [Big Table](labs.google.com/papers/bigtable-osdi06.pdf  "Big Table"). The document is a must-read for all Cassandra (and Big Table as well) users. It describes used data structures and shows how Google uses it for storing page indexes. Once you read it, you'll never look at Google search engine the same way.

The second position on my list was [Cassandra's wiki page](http://wiki.apache.org/cassandra/ "Cassandra's wiki page"), updated with a speed of light, with plenty of links for topics like eventual constistency. Believe me, you can spend at least a few days going deeper and deeper into more cassandranic state of mind.

The nicest inverted index picture, describing the whole idea can be found in [here](http://www.royans.net/arch/cassandra-inverted-index/ "here"). The post additionally describes the performance of inverted indexes in Cassandra's databases. Probably it was the moment, when I asked myself, is there any mapper, allowing with ease save entities (for DDD fans: aggregate roots), like

[sourcecode language="csharp" light="false"] session.Save(user);```

which automatically will disassemble the entity and store properties, marked as indexed, in inverted indexes.

I found none.

The Deiphobus was born.
