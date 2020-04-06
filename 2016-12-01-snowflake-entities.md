---
layout: post
title: "Snowflake entities"
date: 2016-12-01 00:55
author: scooletz
permalink: /2016/12/01/snowflake-entities/
image: /img/2016/11/stocksnap_0bc7ylpwmi.jpg
categories: ["DDD", "Design"]
tags: ["design"]
imported: true
---

### **TL;DR**

Much to often we incorporate the fallacy of grasping it all the next time. We say, that now, after absorbing some knowledge we will KNOW it all and design the system THE RIGHT way. Unfortunately, even using good methods like the strangler pattern, it's simply impossible to design in a right way that covers the WHOLE domain of a company. Are there any design or architectural patterns that can be helpful? It there a way to make the uncertainty play by our rules?

### Identity

The most important part of any design is an identity. I don't mean the identity from the identity management point of view. I mean the identity that makes the entities, aggregate roots different, the good old-fashioned *Id*. It's quite to common to find entities' properties pointing to another context/domain. A User will have an employee's identifier, a car in an insurance company will be *referenced* in accounting and other contexts. Basically, it will spread its *carId* across the whole company. Have you ever encountered *ThisExternalId* in your system? I bet you have. It's time to end this.

### **One thing, multiple views**

It's not a car that is referenced in the accounting. It's not a user that is referencing an employee. It's the same entity spread across different contexts. Let me give you an example.

The same entity, depending on the context, can have a different meaning. A car in an insurance company will be seen in different dimensions. In some contexts it won't occur at all (mortgage insurance), in others, it will be present. What's the MAIN context? Can one tell what is it? Again, I bet noone can.

It's time to end this referential wars. A car is not referenced by this or that context. None of them requires to have a car identifier as a foreign key. Everyone requires to have an id which can have the **SAME VALUE** in different context. Why? Because the same thing will be understood differently in different contexts! When signing an insurance, it will be just a thing. When paying for an accident it will be another, but after all, it's the same car.

The same thing can be seen in many contexts. In each of them, it will have a separate set of properties that is unique, but will share one and only one property - the identity. The identity will be probably artificial and uniquely generated (you don't want to have duplicates, do you?). A perfect match for that would be GUID or UUID.

### Snowflakes

When asking what a car is, you could imagine asking all the contexts for the same identifier. Getting an insurance info

GET /insurances/<span style="color:#ff0000;">38e5c55b-1b44-4bdc-bd9e-632580736f22</span>

Getting the mailing info

GET /mailing/<span style="color:#ff0000;">38e5c55b-1b44-4bdc-bd9e-632580736f22</span>

And so on and so forth. Getting an empty response from a context means nothing but the fact, that an entity does not exist in a specific context. You can see this as a snowflake, which consists of the center which is an identifier, and arms which are responses from different contexts. None of the contexts creates the entity on its own, but every single one contains a meaningful information about the snowflake.

### Further modelling

It's quite easy to follow this rule and imagine adding another context to a system like this. You don't have to add foreign keys or make changes to other components. You just map a new context with the same bluntly simple rule: share just the identity. After all, if another context wants to handle this entity in some way, it will just use this single meaningful property of an entity, the identity of the snowflake.

I hope that this article shed some light on this modelling technique and showed itself as a solid and an extensible approach towards modelling your domain.
