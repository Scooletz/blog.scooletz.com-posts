---
layout: post
title: "REST subresources as commands"
date: 2016-03-20 20:00
author: scooletz
permalink: /2016/03/20/rest-subresources-as-commands/
categories: []
tags: ["REST", "subresource"]
imported: true
---

This post is written write after reading the [@liveweird](https://twitter.com/liveweird) entry about [How not to hurt yourself while building a RESTful API](http://no-kill-switch.ghost.io/how-not-to-hurt-yourself-while-building-restful-api/). If you don't know it, it's probably a good time to read it.

Sebastian mentions that POSTs can be used for any non-crud like operation. I agree with that as the POSTs have a notion of general commands, they are not idempontent and they actually cause a change in a resource. They're much smarter than DELETEs (just trash it) or PUTs (just replace it with my content). How do you distinguish different operations though? Is a header a good place to put the name? Or maybe a property in a JSON payload?

### Subresources to the rescue

The easiest way to model actual non-CRUD commands I've found so far is modelling the commands as subresources specialized in handling them. For example consider the following resource:

> /car/1

One could argue how to model operations like *rent*, *return*, etc. My take on this would be POSTs against the following subresources:

> /car/1/rent
>
> /car/1/return

This modelling is easy to navigate (either you want to [HATEOAS ](https://en.wikipedia.org/wiki/HATEOAS)or simply [Swagger](http://swagger.io/) your API) and easy to follow. It has an additional advantage as well. It's very easy to bind specific commands to an MVC/WebAPI controller methods as every operation can be handle with a separate method. Even if you write a custom class for each of this operations (a command object), leveraging this, can spare you some time on writing your own routing/dispatching of the request.
