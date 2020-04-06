---
layout: post
title: "Deiphobus, first level caching"
date: 2011-07-15 11:00
author: scooletz
permalink: /2011/07/15/deiphobus-first-level-caching/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "NoSql"]
imported: true
---

Another feature of [my mapper](https://github.com/Scooletz/Deiphobus "Deiphobus") for Cassandra database, is the first level cache, commonly known as the [identity map](http://martinfowler.com/eaaCatalog/identityMap.html "Identity map"). Consider loading an entity of a user type only for displaying its login in the page header. Next, you'd like to get the same property to display it in several places across the page (for instance an author of a posts). I assume, that you use DI and your app creates only one session object per web request, injecting it in all the needed places. Consider the following code:

[sourcecode language="csharp" light="true"]
var user1 = session.Load<IUser>(5);
var user2 = session.Load<IUser>(5);
var user3 = session.Load<IUser>(5);
```

Now try to *ReferenceEquals* all the returned instances. Yes, that's the same object! It means that there will be no problems with dirty tracking (*what instance should I persist?*). Also once an user is loaded with a session object, it can be re-retrieved without db hits. Isn't it nice?
