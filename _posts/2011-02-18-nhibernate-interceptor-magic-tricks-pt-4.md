---
layout: post
title: "NHibernate interceptor magic tricks, pt. 4"
date: 2011-02-18 01:58
author: scooletz
permalink: /2011/02/18/nhibernate-interceptor-magic-tricks-pt-4/
categories: ["NHibernate"]
tags: []
imported: true
---

It's the very last but one entry of NHibernate IInterceptor series (the first three: [1](http://blog.scooletz.com/2011/02/03/nhibernate-interceptor-magic-tricks-pt-1/), [2](http://blog.scooletz.com/2011/02/04/nhibernate-interceptor-magic-tricks-pt-2/), [3](http://blog.scooletz.com/2011/02/14/nhibernate-interceptor-magic-tricks-pt-3/)). We traveled through a few first methods of IInterceptor. It's time to end this travel.

### Am I transient enough?

Being transient in NHibernate world means nothing more than being not-persistent ever. The entity which you create with your *new* operator is transient till you call *ISession.Save(object entity)* method onto it, hence making it persistent. The persistent entities are being tracked for changes, their state is being saved/updated. Creating a new entity object not attached to the session is nothing more than creating a new object. It'd be simply garbage collected if no attaching to the session, via the mentioned method, occurred. How NHibernate recognizes if an entity is transient? It uses a static (ugh, yeah, the static method!) *ForeignKeys.IsTransient* method, which first of all, delegates its execution to *IInterceptor.IsTransient*. If you find a more elegant and intelligent way of resolving 'being transient' than calling *IEntityPersister.IsTransient* here you are your entry point. I did not find it useful in my whole journey with NHibernate.

### I am collection and I need some action to be performed onto me

The following three methods of IInterceptor:

* void OnCollectionRecreate(object collection, object key)
* void OnCollectionRemove(object collection, object key)
* void OnCollectionUpdate(object collection, object key)

are called during flushing collections, when a session flush occurs. The first step of flush is flushing of entities, the second - flushing collections. When the collection flush occurs, for each collection, it is determined what kind of action should be executed against this collection. The three choices matches the names of the IInterceptor methods. It's worth to know and read something more about *IPersistentCollection* interface which an implementation instance is passed as the first parameter. As it was in the *IsTransient* case, I did not use any of this methods either.

### I am being deleted!

When a delete occurres, the very first thing NHibernate does, is calling *IInterceptor.OnDelete(object entity, object id, object[] state, string[] propertyNames, IType[] types)* method. It does not provide any flow control and is simply called by the default *IDeleteEventListener* before scheduling the real delete action. What one can do in here, is checking whether an entity can be deleted and, for instance, throw an exception if this operation is not permitted.

In the last entry I will provide the description of the last, few methods as well as an example of full-blown implementation of IInterceptor.
