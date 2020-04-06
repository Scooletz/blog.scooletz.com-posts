---
layout: post
title: "NHibernate interceptor magic tricks, pt. 5"
date: 2011-02-22 23:15
author: scooletz
permalink: /2011/02/22/nhibernate-interceptor-magic-tricks-pt-5/
nocomments: true
categories: ["NHibernate"]
tags: []
imported: true
---

The previous parts of NHibernate interceptor journey can be found here: [1](http://blog.scooletz.com/2011/02/03/nhibernate-interceptor-magic-tricks-pt-1/), [2](http://blog.scooletz.com/2011/02/04/nhibernate-interceptor-magic-tricks-pt-2/), [3](http://blog.scooletz.com/2011/02/14/nhibernate-interceptor-magic-tricks-pt-3/), [4](http://blog.scooletz.com/2011/02/18/nhibernate-interceptor-magic-tricks-pt-4/). In this last entry I will describe all methods not covered previously. The example will be added as a separate entry as it needs a few more tests to be done, but hey! Now you'll know something about all methods of IInterceptor! So, let's start!

### When dirt is flushed

When a session is flushed, there is a dirty check which is performed for each entity stored in the first level cache (in other words: a session cache or a persistent context). The dirty check is done by raising a *FlushEntityEvent*. The event object contains all the needed information about entity and is processed by a chain of event listeners. There is one default event listener for this type of event, called *DefaultFlushEntityEventListener*. It's the one which is responsible for checking whether a loaded entity needs an update. It's out of scope of this post to go deeper into NH event model, hence finally, when an entity is marked as to be updated, an update action is created. During this process, under certain conditions,
*bool IInterceptor.OnFlushDirty(object entity, object id, object[] currentState, object[] previousState, string[] propertyNames, IType[] types)* method can be called. I tend to use event listeners rather than using this method, not dealing with flushing.

### Lock 'N Load!

This one will be pretty straightforward: *bool IInterceptor.OnLoad(object entity, object id, object[] state, string[] propertyNames, IType[] types);* is called each time an entity object is being preloaded (passing an event through all IPreLoadEventListener). There are two cases when this happens: an entity is being assembled from cache and when an entity is being just loaded from its db representation. The state passed as the table of objects, is the state retrieved from db/cache (the representation is the same: an array of objects), and if modified, will affect setting the properties of your entity. For instance if we change *state[i]* which represents a property of name *propertyName[i]* and is of type *types[i]*, it will cause this property to set to the value. Even more, the new value will be saved as the original state of an entity and the dirty check will not mark an entity as *dirty*.

### Format me this, format me that

 *SqlString OnPrepareStatement(SqlString sql)* allows you to change the sql string which is being sent to the db. You may replace some parts, as well as create an interceptor which will write the whole SQL of your web app to the trace. It's also the easiest way to check how many times you app hits db (if an interceptor is created and attached on per session basis and the session is created once, per request, then you can easly count SQL statements executed per request). The other, much nicer and elegant, way of logging all statements is attaching an appender to the right *ILog*. BTW, this is what NHProf does!

### On save, save?

*bool OnSave(object entity, object id, object[] state, string[] propertyNames, IType[] types)* method is called when an entity is saved. As in many cases, the last three tables contains: object properties values, property names, types of the properties. The method returns bool, which indicates whether values from the *state* should be copied onto object properties. If you return false, although the state is changes, NHibernate will not be notified to copy these values. I tend to use proper event listeners *IPreInsertEventListener* and *IPreUpdateEventListener* rather then making IInterceptor more rigid.

### Preflush, postflush, inflush, outflush...

*void PreFlush(ICollection entities);* and *void PostFlush(ICollection entities);* methods do what they destined to do. The first is called with an collection of all entities which state is going to be changed during the flush which is comming, the second, is called with an collection of all entities which state was flushed already. Want to customize logging for all objects flushed (NHibernate does it already!)? That's the place you should go!

An example will be provided soon!
