---
layout: post
title: "Deiphobus, it simply works with your Cassandra"
date: 2011-07-05 10:30
author: scooletz
permalink: /2011/07/05/deiphobus-it-simply-works-with-your-cassandra/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "NoSql"]
imported: true
---

I've migrated [Deiphobus](https://github.com/Scooletz/Deiphobus) to the GitHub. Then, I refactored it a lot, or maybe better - rewrote. Now it became fully functional wrapper for [Cassandra](http://cassandra.apache.org/ "Cassandra db"). The newest version pushed recently closes a major list of features I wanted to add. If you want to use Cassandra from your .NET code, I strongly encourage you to take a look into (at least) a [readme file](https://github.com/Scooletz/Deiphobus/blob/master/README) which covers the majority of cases, where you may use it. And the list of features is:

* *unit of work (ISession)*
* *first level caching (identity map)*, which simply does not hit db when entity is requested twice in one session
* *lazy load*, loading an entity does not hit db till you get one of its properties
* *mapping properties groups to column families*, which allows you to load commonly used sets of properties in one db hit
* *prefetching a few entities' data* to omit 'SELECT n+1 problem'
* *entities' references*, as one entity can reference another simply by creating a property of the other type
* *entities' collections*, as one entity can reference another; it's lazy loaded, so adding entities does not load a whole collection from db
* *automatic dirty checks* performed when *ISession.Flush* is called
* the latest feature: *second level cache* for infrequently changing data (column families), caching a part of your entity (for instance a user name and its whole column family) in the memory cache (the default implementation)

I'll describe all of them in the forthcoming entries, as a lot of changes happened since the last Deiphobus post.
