---
layout: post
title: "A story of Cassandra's brother: Deiphobus"
date: 2010-08-19 12:28
author: scooletz
permalink: /2010/08/19/a-story-of-cassandras-brother-deiphobus/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra"]
imported: true
---

The Apache Cassandra Project develops, as it is written on the official page of the project, a highly scalable second-generation distributed database, bringing together Dynamo's fully distributed design and Bigtable's ColumnFamily-based data model (wow!:P). The mentioned properties of this architecture brings profits, but also some limitations and challenges such as:

* no transaction support
* no atomic operations (the data entry can be stored in one node of replicating nodes, the others can still return the dirty-read data)
* possibility of slow 'eventual consistency' (see http://wiki.apache.org/cassandra/HintedHandoff)
* no obvious quering (there is no possibility to simple ask for a song titled 'Octavarium')

Next, the Cassandra client, Thrift-based, provides a very low level API, which does not allow connection pooling, using identity map and many more features so well-known from other db access tools (ADO.NET, NHibernate). According to the Cassandra's list of .NET clients there are three of them:

* Aquiles: http://aquiles.codeplex.com/
* Hector Sharp: http://www.hectorsharp.com
* Fluent Cassandra: http://github.com/mana

Although they provide a nice object wrap around the Thrift interface (or LINQ in Fluent Cassandra), they lack few features. These deficiencies brought to life an idea of Deiphobus, the more advanced access API for Cassandra database. The core features of Deiphobus will be:

* unit of work pattern
* identity map
* automatic inverted index creation (based on a fluent configuration)
* compensation strategies (for the transactionality lack)
* future queries (the very same as in NHibernate)

In the next post I'll describe the influences and basics of Deiphobus design.

Take care
