---
layout: post
title: "Back to relational, hibernating world"
date: 2010-08-26 20:00
author: scooletz
permalink: /2010/08/26/back-to-relational-hibernating-world/
nocomments: true
categories: ["NHibernate"]
tags: ["testing"]
imported: true
---

Recently, I've been extensively using [Fluent NHibernate](http://fluentnhibernate.org/ "Fluent NHibernate").  As always, setting up the tests made me setup the SqlLite to cope with NHibernate. I found that FNH provides a simple class with a mouthful name *FluentNHibernate.Testing.SingleConnectionSessionSourceForSQLiteInMemoryTesting*. All it does is accepting the *FluentConfiguration* object and providing the created session for your tests. Nice :]
