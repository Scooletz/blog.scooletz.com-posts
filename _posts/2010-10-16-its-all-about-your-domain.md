---
layout: post
title: "It's all about your domain"
date: 2010-10-16 20:00
author: scooletz
permalink: /2010/10/16/its-all-about-your-domain/
nocomments: true
categories: ["Architecture", "Design patterns"]
tags: ["architecture", "authorizations", "design patterns"]
imported: true
---

I dislike libraries which make me pollute the domain with any implementation and reference. I made up my mind about Log4Net and NHibernate, and now I'm totally cool to add the naked (not wrapped in a _super_abstracted_layered_total_wrap_ ) reference to my project and query the NH's session treating it as the repository. What I dislike is when the friction between the library and application makes me leak its interfaces all around or wrap its plenty of interfaces.

The [Themis](http://themis.codeplex.com/ "Themis") is about providing an (in majority of cases) authorization system, with almost no overhead. In the last part I mentioned the three main parts of the Themis. I'll start with the last, the most meaningful - your domain. Imagine a very complex system with plenty of roles. Imagine that each role can have a different context for different users, for instance:

* Bob, has roles:

    *   Administrator, with properties

    *   SkillLevel: 5

    *   Permissions: Permissions.Max

    *   Worker, with properties

    *   Floor: 4

    *   Worker, with properties

    *   Floor: 5
* Alice, has roles:

    *   Manager, with properties:

    *   MobbingFloor: 4

    *   Manager, with properties:

    *   MobbingFloor: 7

    *   Mobbing

As you can see roles, for me are not only simple strings, but rather containers for contexts and they should be included in your domain. Using NHibernate I'd create a base class Role and derive other roles from it. Additionally, I'd add a set property to the User, simply called Roles and that would be it. Going back to the contextful roles, it allows you to incorporate them into more meaningful statements like "Administrator can Manage only Programs with RequiredSkillLevel lesser then his/her". A simple business rule having a few parts: the privilege itself - Manage, the subject of rule - Program, and the condition. Having the roles implemented in the way described earlier, I'd be very pleased if I could simply write a lambda expression having Role and Program as parameters and checking the relation between skill levels. I'll write them in the next post.

To sum up: it's all about your domain, having roles as nonempty entities with connected to the aggregate, the user.
