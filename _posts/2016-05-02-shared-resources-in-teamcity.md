---
layout: post
title: "Shared Resources in TeamCity"
date: 2016-05-02 09:00
author: scooletz
permalink: /2016/05/02/shared-resources-in-teamcity/
categories: []
tags: ["shared resources", "TeamCity"]
imported: true
---

It's a common requirement that a set of your tests depends on some resources. It might be a database or an Azure Storage account. It's possible that instead of providing TeamCity with an administrator account (giving a subscription access for Azure) you'd prefer to have a limited preexisting set or resources like databases or Azure Storage accounts that are leased for a build time by a particular agent. As soon as build is finished the resource would go back to the pool to be leased for another build.

Fortunately TeamCity has a built in ability for this purpose called [Shared Resources.](https://confluence.jetbrains.com/display/TCD9/Shared+Resources) This can be defined on any project level and used as a parameter of any build configuration below. Shared Resources feature provides you with all the capabilities mentioned before, removing all the burden of managing a resource pool. In the same way a build leases an agent, an agent leases a shared resource. Nice, simple, easy.
