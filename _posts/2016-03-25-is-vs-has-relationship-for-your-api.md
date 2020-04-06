---
layout: post
title: "IS vs HAS relationship for your API"
date: 2016-03-25 10:00
author: scooletz
permalink: /2016/03/25/is-vs-has-relationship-for-your-api/
nocomments: true
categories: ["DDD", "Design"]
tags: ["Aggregate", "composition"]
imported: true
---

There is an urge of making things automagically. For instance, when you have a DDD Aggregate, one could consider automatic publishing all the commands as the service API. As The Aggregate is a part of a model & a language which you agreed to use, that seems to be a perfect match to **be** your API. Is it?

### IS vs HAS

There is a rule of [Composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance). It says that instead of deriving from different components, one should compose bigger parts from already existing by using them, but no deriving. A good example might be a user and an employee. As an employee, you are given a user in the system. You might try to model it with the derivation in mind. Is an user an employee as well or the other way around? There's no good answer to this.

You could model it in another way. There an employee, that a user has access to. When a user logs in he/she can access data of an employee is attached to him/her. You can see where is it going. Keep things minimal, use other elements but do not introduce a relation of being something.

### A bit abusive allegory

Now ask yourself a question. Is your API using the model or is it the model? In the majority of cases, the interfaces of your API & your model may be aligned, but they are not the same! Even if you publish operations named after a model that you established, you'd like your API to use the model just in case of remodeling the domain. It's good to automate and do not write much code. On the other hand, it's good to have proper abstractions separating concerns of two different worlds.
