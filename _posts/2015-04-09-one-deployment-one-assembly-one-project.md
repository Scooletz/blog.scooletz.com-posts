---
layout: post
title: "One deployment, one assembly, one project"
date: 2015-04-09 11:00
author: scooletz
permalink: /2015/04/09/one-deployment-one-assembly-one-project/
categories: ["Architecture", "Design"]
tags: ["design", "design by feature"]
imported: true
---

Currently, I'm working with some pieces of a legacy code. There are good old-fashioned DAL, BLL layers which reside in separate projects. Additionally, there is a *commo**n* project with all the interfaces one could need elsewhere. The whole solution is deployed as one solid piece, without any of the projects used anywhere else. What is your opinion about this structure?

To my mind, splitting one solid piece into non-functional projects is not the best option you can get. Another approach which fits this scenario is using feature orientation and one project in solution to rule them all. An old, the deeper you get in namespace, the more internal you become, is the way to approach feature cross-referencing. So how would one could design a project:

* /Project

    *   /Admin

    *   /Impl

    *   PermissionService

    *   InternalUtils.cs

    *   Admin.cs (entity)

    *   IPermissionService

    *   Notifications

    *   /Email

    *   EmailPublisher.cs

    *   /Sms

    *   SmsPublisher.cs

    *   IPublisher.cs

    *   Registration

    *   ...

I see the following advantages:

* If any of the features requires reference to another, it's an easy thing to add one.
* There's no need of thinking where to put the interface, if it is going to be used in another project of this solution.
* You don't onionate all the things. Now, there are top-bottom pillars which one could later on transform into services if needed.

To sum up, you could deal with features oriented toward business or layers oriented toward programming layers. What would you choose?
