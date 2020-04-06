---
layout: post
title: "NHibernate's No-Proxy Laziness"
date: 2011-04-06 20:00
author: scooletz
permalink: /2011/04/06/nhibernates-no-proxy-laziness/
nocomments: true
categories: ["DynamicProxy", "NHibernate"]
tags: []
imported: true
---

It's been a while since last post, but I'm alive and still kicking, so let's go deeper into another aspect of one of my most favorite libraries. Today's entry is all about lazy-loaded properties.

### The setup

Whether you use FluentNHibernate with its property of PropertyPart, allowing you to choose a property's laziness, or you still use xml description of your entities, you've got three choices for setting the laziness of your *to-one* (*many-to-one*, *one-to-one*) properties, with the default set to the second:
* false
* proxy
* no-proxy

### The performance

The very first one is easy to understand: do not lazily load this property. I want it always, I don't care about all the stuff with getting to much data, possibly fetching the whole database into my session (if you can traverse a whole mapping graph from your type, that might happen). Use it wisely and sparingly, only when you know that *every* time you get the one object, you want to have another.

The second is a default one, NHibernate takes care of you, getting only one entity at time. It's good, allowing you to set fetches explictly every time you need in your query (IQueryOver, ICriteria, IQuery, etc.) The problem is, when a type of a specified property has any derivations and this leads us to the third option: having your properties lazily loaded with their types being not proxified at all.
The whole situation is well described in [Ayende's post](http://ayende.com/Blog/archive/2010/01/28/nhibernate-new-feature-no-proxy-associations.aspx). When, for instance, you three classes mapped: A, B (which derives from A) and X, and X has a property of type A, after having this property lazily loaded you cannot cast this to B, hence, the type being hidden behind this property is a proxy type, let's call it PA. It's obvious, since in the .NET there is no multiple inheritance, the proxy of A (PA : A) cannot derive from B as well. Once you set the no-proxy on your property, the problem seems to be gone. But how? What's happened?

### The prestige

The property of type A has been altered. Every time it is called, it is not a simple getter. It's been wrapped to return A, but with loading it on demand, when a property is called for the first time. How it might happen? The class X is not a class X. It's a proxy (PX), with all the properties' types preserved, hence, your type checking or casting A to B works. The thing can get a bit more complicated if you want to serialize X to JSON and return it as a action result (PX is not the best serialization candidate; proxy adds *a few* bonuses) or downcast X, to another class. Although it seems to be very powerful, use it sparingly to not be overwhelmed with having everything proxyfied.
