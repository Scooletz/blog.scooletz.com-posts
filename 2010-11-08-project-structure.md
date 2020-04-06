---
layout: post
title: "Project structure"
date: 2010-11-08 20:00
author: scooletz
permalink: /2010/11/08/project-structure/
nocomments: true
categories: ["Architecture", "Design"]
tags: ["architecture", "design", "VS solution"]
imported: true
---

I've been working on a few quite big projects recently. The first I was dropped into was designed a few months before my arrival and had the solution consisting of almost 60 projects. The references were cascading, with each project having all the previous referenced. The web project depended on all of the projects, and had nothing in common with terms like *dependency injection*, *ORM*, *best practices*. All the data access code was written with using *SqlCommands* with caching thrown in multiple places. The same with other blocks you can imagine such as validation, logging, gathering statistics, etc. Reviewing the code made me think about [NIH syndrome](http://en.wikipedia.org/wiki/Not_Invented_Here "http://en.wikipedia.org/wiki/Not_Invented_Here").

The second project was designed to handle a lot of stuff, for instance lifetime management (a very special case) and  very domain specific workflows. The design was done in 80% when I entered the project. We tried to write not much code, using already provided frameworks for different purposes like: NHibernate, log4net, NServiceBus, Irony parser (for domain specific language). Then the first version was established, the solution had 16 projects. Two weeks ago I made a major refactorization, reducing a few service and code-based (*yet another abstraction*) boundaries, creating a more integrated, yet _not_more_ coupled system. The final solution was performing better and had much simpler deployment. It consisted of 11 projects.

The very last solution consisted of 6 projects:

* Web (having reference to NHibernate and Model)
* Web.Config (infrastructure stuff, all the DI registrations, referencing all libraries and setting up controller factory in ControllerBuilder and default ModelBinder, infrastructure service implementations)
* Model (entities, factories, services, types - like email, domain events and their handlers)
* Model.Impl (implementation of interfaces from Model split in the Model's manner)
* Model.Persistence (all the NH event listeners, Fluent NH mappings, Solr services)
* Model.Presentation (simple denormalized readonly objects to be used for queries result, possibly with Solr)

Do my solutions become to minimal? As far as the last solution is being built it does not seem so. The responsibilities are clean and all can be easily put into this separation. How do you find it? Any ideas?
