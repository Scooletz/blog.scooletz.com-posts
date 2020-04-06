---
layout: post
title: "NHibernate EntityAwareModelBinder for ASP MVC"
date: 2011-05-23 11:00
author: scooletz
permalink: /2011/05/23/nhibernate-entityawaremodelbinder-for-asp-mvc/
nocomments: true
categories: ["ASP MVC", "NHibernate"]
tags: ["C#"]
imported: true
---

It's time to provide some more features before getting deeper into detached actions for ASP MVC. The very first requirement is to have a nice model binder, which not only uses a container to resolve all the parameter types ([my previous model binder](http://blog.scooletz.com/2010/11/11/asp-mvc-model-binders/ "ASP MVC model binders")), but also is intelligent  enough to deal with an ORM of your choice. And mine is NHibernate.

Before getting the final snippet, a few assumptions has to be made:

* No hierarchies are bound automatically. For sake of simplicity, we're considering only simple entities with no derivations.
* All the collections, which reflect one-to-many relationships, are bound in the inverse way. The *one* ending has its collection marked as inversed (there is a drop-down for the *many* ending, where the parent can be selected)

To provide you with a better way of viewing the code, I pushed the file to the gisthub and it can be located under: [https://gist.github.com/984310](https://gist.github.com/984310).  It's good time to take a look into it.

As you can see, the binder does some pretty nice things:

* It still uses container as the fallback for all the models. Using Windsor you'd better be sure to have all of them registered
* If entity has its id passed, it uses session to get it; otherwise a new instance is created. It's up to you to save it in your session

Maybe it addresses a few concerns, like handling entities and non entities in one binder, but it'd be easier to get the whole idea having one straight class without the whole projects.

This is my default model binder in my current project so far and it provides strong base for the detached actions mechanism, which will be covered in the very next entry.

Thoughts?
