---
layout: post
title: "I want my own db"
date: 2011-01-29 20:00
author: scooletz
permalink: /2011/01/29/i-want-my-own-db/
nocomments: true
categories: ["NoSQL", "Optimization"]
tags: ["NoSql"]
imported: true
---

Writing your own db is getting more and more popular, isn't it?:] Especially, if you want to create your custom, NoSQL solution. It's worth to mention that even [Ayende](http://ayende.com/Blog/default.aspx), on of the core NHibernate contributors commited one. It's called [Raven DB](http://ravendb.net/) and is a document database based on a managed storage with [Lucene](http://lucene.apache.org/lucene.net/) included for querying purposes. One of its interesting attributes is that Lucene's index is eventually consistent with the managed storage (you can wait a dozen of miliseconds before your updated document will be indexed and searchable) . If you want to learn sth more about NoSQL solutions and have a plenty of time for watching interesting interviews, you should visit [NoSQL Tapes](http://nosqltapes.com/). They're definitely worth to watch.

Speaking about 'your own db', it's worth to mention, that sometimes NoSQL dbs has dramatically different paradigms. I cannot imagine easily switching between Cassandra and MongoDB, or between Redis and RavenDB. They abilities do not map simply. Choosing one should be done with a deep view into your system requirements (for instance Twitter perfectly fits the Cassandra approach).

After this short introduction I wanted to present results of different strategies of aggregating (summing) one column of 100000000 rows with a spike code written recently. The code itself was written after rereading Google's article about Dremel as well as [What every programmer should know about memory](http://www.akkadia.org/drepper/cpumemory.pdf) and a plenty of [Joe Duffy](www.bluebytesoftware.com) stuff. The result is quite nice in my opinion and is represented with

### 0,098273895 seconds

It's quite qood, isn't?

The spike I wrote is a journey, not a way of accomplishing something. With [Themis](http://themis.codeplex.com/) there was an aim and reason, with this, at least for now, it' only play.
