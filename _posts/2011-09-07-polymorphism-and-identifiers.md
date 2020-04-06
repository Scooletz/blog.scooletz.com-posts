---
layout: post
title: "Polymorphism and identifiers"
date: 2011-09-07 20:00
author: scooletz
permalink: /2011/09/07/polymorphism-and-identifiers/
nocomments: true
categories: ["Design", "NHibernate"]
tags: ["CombGuid", "HiLo", "IIdentifierGenerator", "Polymorphism"]
imported: true
---

Polymorphism makes you use truly unique identifiers. If you don't agree, consider the following scenario:
![Polymorphism diagram](http://yuml.me/diagram/scruffy/class/ [IEntity|Id]^-.-[IAsset|Name], [IEntity|Id]^-.-[ITaggable|Tags], [IAsset|Name]^-.-[Document|+Name;+Tags], [IAsset|Name]^-.-[Briefcase|+Name], [IAsset|Name]^-.-[Box|+Name], [ITaggable|Tags]^-.-[Document|+Name;+Tags], [Briefcase|+Name]->[Document|+Name;+Tags] [Box|+Name]->[IAsset|Name]) Imagine, that client wants you to create a registry (list view, whatever), that will enable him to query assets and apply different criteria onto them (there is not much in the interface, but name can also be queried with SQL *like* operator). You immediately notice, that you can use polymorphic queries from *NHibernate*, which spans to all implementations of the given interface. They can be some problems with the queries efficiency (paging is done in the memory), but skipping this fact, you've done it. You have your list of assets presented to the user, who instantly notices that there are a few rows with the same id. *What's going on*, you ask. Ahhh, you're still using the old-fancy db identity identifier generator. Not only it's generates unique id per table, but also removes all the bonuses connected with batching inserts. How can you overcome such an obstacle?

The solution is simple: use GuidGenerator, especially *CombGuidGenerator*, as your *IIdentityGenerator*. If your customer hastes these 128-bit long chains, use HiLo algorithm, which stores within the process a package of identifiers ready to be used. Once they're used, it goes to the db to transactionally get another one. Span this HiLo for all tables/entities hierarchies which can be queried in a polymorphic way. For the best result, set it for *all* entities' types, and enjoy having unique long/int identifiers across your whole database.
