---
layout: post
title: "Deiphobus, no more SELECT n + 1"
date: 2011-08-04 11:00
author: scooletz
permalink: /2011/08/04/deiphobus-no-more-select-n-1/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "performance"]
imported: true
---

The previous post contained an information about lazy loading of group of properties, let's call them families as it is called in the Cassandra. What about the following code. How many db hits you'd like to get by default?
[sourcecode language="csharp" light="true"]
using (var s = sessionFactory.Open())
{
	var user = s.Load<IUser>(5);
	foreach(var post in user.Posts)
	{
		Console.WriteLine(post.Title);
	}
}
```
I'll tell you how many you'll get. The answer is two: first hit will occur, when a collection of posts is accessed in the *foreach* loop, the second - when a title is printed on the console. During the second hit all the posts loaded in the session will have their titles loaded. In some cases it may drive to a small overhead, but it simplifies batching and working with your entities in the majority of cases. Would anyone like to set *FetchMode*, like it was done in the NHibernate? ;)
