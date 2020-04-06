---
layout: post
title: "Composition vs derivation"
date: 2011-05-09 20:00
author: scooletz
permalink: /2011/05/09/composition-vs-derivation/
categories: ["Architecture", "Design", "Design patterns"]
tags: ["architecture", "ASP MVC", "design", "design patterns"]
imported: true
---

Assume you're writing some reports in your application. You've just created the third controller covering some kind of reporting and it seems to be, that all the three controllers have a very similar  code, modulo type passed as the entity type to your NH ISession.QueryOver() or another data source. It seems that, if the method creating report base was generic, it could be used in all the cases. You want to *extract* it, make it clearer and to stop repeating yourself. What would you do?

### Derivation

The very first thing is to *extract method* in each of the controllers. Now they seem almost the same. A place is needed where the method can be easily *moved*. How about a super type? Let's create a controller, call it in a fashionable way, for instance: ReportControllerBase, and move the method in there. Now you can easily remove the methods in the deriving controllers. Yeah! It's reusable, everyone writing his/her report can derive from the ReportControllerBase and use its methods to speed up his/her task.

### Composition

The very first step is exactly the same: the  *extract method* must be done, to see the common code. Once it's done, you notice that the whole method has only a few dependencies which can be easily pushed to parameters, for instance: isession, the entity type passed to query, the value used in some complex where clause, etc. You change all the field and properties usages to parameters which allows this method to be static. You create a static method and turn it into an extension method of a session. The refactorization is done, you can easily call this method in all the controllers, by simply calling an extension onto a session.

### What's the difference and why composition should be preferred

If you use C# or Java you should always be aware of one limitation: you can derive from only one type. Spending this 'once in a lifetime' for a simple functionality extraction, for me - that's the wrong way. Using composition, and deriving only when you truly see that one type *is* another type, that's the right way to go. In the next post I'll write about *ASP MVC Detached Actions*, a simple mechanism you can use, to derive your controllers only, when it is needed and delegating common actions, without it.
