---
layout: post
title: "How to guide users of your API"
date: 2011-01-22 20:00
author: scooletz
permalink: /2011/01/22/how-to-guide-users-of-your-api/
nocomments: true
categories: ["C#", "Design"]
tags: ["design"]
imported: true
---

After winter holidays it was time to get back to the reality and to make up for the pleanty of unread posts in my reader. One of the read posts was [Christopher Bennage's](http://devlicious.com/blogs/christopher_bennage/archive/2011/01/21/disabling-certain-linq-operations.aspx) entry about RavenDB API. The main problem with API was that it allows calling every Linq extension method on the Raven session. Even, the unwanted one - *ToList*. Christoper's proposal is to provide an obsolete ToList method, which accepts the Raven query interface, and by being obsolete informs about the best way of getting a list of objects.

Simple, nice, powerful.
