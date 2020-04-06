---
layout: post
title: "A CRUD action provider"
date: 2011-06-20 11:00
author: scooletz
permalink: /2011/06/20/a-crud-action-provider/
categories: ["ASP MVC", "C#"]
tags: ["ActionInvoker"]
imported: true
---

As it was described in [the previous entry](http://blog.scooletz.com/2011/05/25/an-action-provider-other-than-controller/ "An action provider other than controller"), the idea of the action providers is to allow to gather cross-cutting concerns into more generic (not in the C# terms, but functionalities/features) bag than a derivation from a generic controller. To provide a working example, a standard CRUD operations can be taken into the consideration. Let's take a look into another [gist](https://gist.github.com/1033019 "CRUD action provider").
I do hate static code, but for sake of a simplicity all the CRUD operations were moved to a static class, *CrudActions*. All of them are pretty simple, as they use an entity model binder introduced in the former blog entries. For instance, when an entity edit result is posted to the server, a model binder will take care of updating the entity, so the entity being passed to the *[HttpPost]Edit*  will be already altered with the changes made by a user.

The very second step was discovering all of the CrudActions. It is made by a DetachedActionProvider which caches information in a ReadWriteCache taken from the MVC internals. To close the open generic of CrudActions, an entity type with an id's type is needed. The entity type information is retrieved from a *IEntityController*, which is a simple markup convention (you can come up with a name convention, or delegate it somewhere else), for attaching to a controller a handled entity type. The id metadata are gathered from the NHibernate's *ISessionFactory*. Once the data are retrieved, the CrudActions generic class definition can be turned into a closed generic and the specific method can be found.
Wait a second... how do we transform a MethodInfo into an *ActionDescriptor*? First, the method infos are wrapped with an ActionInfo instance (it caches attributes as well as the method info itself). Next, the ActionInfos, those who applies (HttpGet, HttpPost, etc.), are passed to the ReflectedMvc to create a ReflectedActionDescriptor.
At the end I left some bitterness about MVC internalization policy. I must admit, that I don't understand internalizing types or constructors or any members which could be helpful for future app development. That case applies to the ReflectedActionDescriptor ctor, which (the public one) throws an exception if the method passed to it is a static method. The other, non-checking one, is of course internalized. I'm asking: why? The internalization should be used as a last resort, no as a 'hide it, because I cannot predict if someone doesn't want to crush MVC in his project' rule.

Hope you like the action provider extension point and will use it for cross-cutting, 'I can compose, not only derive from a generic class' actions in your controllers.
