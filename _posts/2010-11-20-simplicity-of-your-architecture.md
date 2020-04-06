---
layout: post
title: "Simplicity of your architecture"
date: 2010-11-20 21:54
author: scooletz
permalink: /2010/11/20/simplicity-of-your-architecture/
nocomments: true
categories: ["Architecture", "Design", "Personal"]
tags: ["architecture", "design"]
imported: true
---

A few days ago I watched a nice presentation published by Infoq called [Simplicity Architect](http://www.infoq.com/presentations/Simplicity-Architect) which made me think one more about design decisions I've been making through the last one year. Dan North, which was the speaker remind of the major problem every architect, I'd say, designer has. The problem is complexity lurking in the dark corners of each solution. I can remember times when from-zero-to-working-example took my team more than one month because of the db creation, design, and other stuff which you can qualify as things, which your business don't understand. During the very last startup, it took me only 2 hours to provide the very basic set of functionalities with viewing and filtering a list of entities, editing them and so on. I did use a few tools (NHibernate, log4net, Unity and my infrastructure part injecting it all in MVC in my way), but it was only 2 hours to have a working example and it was _simple_ in terms of design. I did like it as the business owner did.

Hope you don't get too complex either.
