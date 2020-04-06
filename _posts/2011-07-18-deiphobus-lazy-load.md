---
layout: post
title: "Deiphobus, lazy load"
date: 2011-07-18 11:00
author: scooletz
permalink: /2011/07/18/deiphobus-lazy-load/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "NoSql"]
imported: true
---

Last time, when an identity map of the [Deiphobus](https://github.com/Scooletz/Deiphobus "Deiphobus") was described, one user entity of was asked about several times. We already know, that the same object was returned, but what about hitting the Cassandra DB? Consider the following code:
[sourcecode language="csharp" light="true"]
var user = session.Load<IUser>(5);
// some other loads and operations
var model = user.Login; // here goes db hit!
```

As you can see, the database is not hit till one of the properties is queried. It's default and only mode of loading entities with *Deiphobus*. Is allows a great reduction of db calls, if your code is structured in a right way (query for data first, then operate). The question is, what if a user holds a few massive, in terms of bytes transported, properties. Will of them will be loaded at once, even if they're unneeded? The answer will be revealed in a very next post.
