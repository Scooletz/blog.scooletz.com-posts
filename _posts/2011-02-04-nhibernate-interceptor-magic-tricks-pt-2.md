---
layout: post
title: "NHibernate interceptor magic tricks, pt. 2"
date: 2011-02-04 20:00
author: scooletz
permalink: /2011/02/04/nhibernate-interceptor-magic-tricks-pt-2/
nocomments: true
categories: ["NHibernate"]
tags: []
imported: true
---

In the [previous post](http://blog.scooletz.com/2011/02/03/nhibernate-interceptor-magic-tricks-pt-1/), I described the basics of NHibernate's IInterceptor as well as using it to manage transaction-connected actions. It's time to move on and know better the other methods of this interface.

### Am I dirty? Yes I am!

The very next method is connected with the term: *dirtyness*. During a flush, which occurs from time to time (it's a topic for another entry, but it's well covered by plenty of other blogs and books), a NHibernate's session performs a check, whether any of persistent objects loaded to the session context had any of its properties changed. Being dirty, especially for an entity, equals having any property changed. It's obvious that running through all of the loaded entities can take time, so that's another reason why you should bother how many objects are loaded during one request/session.
Ok, so what about *IInterceptor.FindDirty* method? When entity is checked for its dirtiness, the very first step is to do it via mentioned method. If the method returns a non null result, an array of ints, then the default check is not performed and it's taken for granted, that properties with the identifiers hold by the result array are dirty. Their values will be used to finally generate the sql update script. If you return null, a standard mechanism will be used.

### Excuse me sir, how can I get the car with this id?

When NHibernate tries to get an entity it goes through the following steps:

1. try to hit session cache (persistence context)
1. try to hit second level cache
1. try to hit database

The interceptor's method *GetEntity* is used between the first and the second step. If the entity was not loaded in previously by this session, the interceptor is asked to retrieve an entity. In majority of cases it does not do it. Then the second cache, if any is hit. What can it be used for? When a session is cleared, the first level cache is dropped. If it occurred frequently, the most commonly used entities could be stored in a session's interceptor, to spare this poor db some hits.

### Say my name, say my name

Entities names are very important part of NHibernate. It uses it with connection of identifiers to create a truly unique entity id (yep, there are some cases when you don't use Guids). For instance, when you make an object persistent, and call *ISession.Save(object entity)* it's hard to guess which entity do you save. To get this information the method of session is called. The very first step is to delegate it's responsibility to the IInterceptor. If it returns null, the type of the entity is retrieved and is used to resolve the entity's name. What can this method be used for? Imagine that your mapping describes not the class, but an interface. What *object.GetType()* will return? Your interface type? No! It will return the real type of the object, and when it is not mapped, an exception will occur. Having this method, you can easily combine a DI container used with your entities, having interfaces mapped instead of real classes.

### Summing up

Now you know something about NHibernate's *dirty checks*. You can also do some reaaaally interesting stuff with NHibernates caching. Finally, you're partially prepared to introduce dependency injection into your domain mapped with NHibernate. To know how fully inject into your mapped domain, read the very next entry.
