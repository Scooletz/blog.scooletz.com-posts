---
layout: post
title: "Backbone & underscore beauty"
date: 2013-04-22 10:00
author: scooletz
permalink: /2013/04/22/backbone-underscore-beauty/
nocomments: true
categories: ["Javascript", "SPA", "Uncategorized"]
tags: ["Backbone", "javascript", "js", "Marionette", "Underscore"]
imported: true
---

Recently I've joined a project which is based on the Single Page Application paradigm. This means no page refreshes after the first load, after login and a plenty of client side scripting. To enhance the project speed a stack of client side libraries is used including, [RequireJS](http://requirejs.org) (for module management), [Underscore](http://underscorejs.org/), [Backbone](http://backbonejs.org/) and [Backbone Marionette](http://marionettejs.com/). There are more of them, but the last three are very interesting because of their paradigm.

Underscore, Backbone and Marionette are not frameworks. They are libraries, great libraries providing a few tools to deal with a web app development. They do not make you to use all of they <del datetime="2013-04-13T15:54:09+00:00">features </del> components (as 'features' are overloaded, framework term). You can pick wisely and use only a few of the provided tools. Of course there is a risk of creating a monster, an unusable app, but it's up to you and your paradigms, what to use and what to implement on your own. Strongly encourage you to get accustomed to them, especially all of them have a beatiful common denominator, they provide the annotated codebase ([code as documentation](http://lostechies.com/derickbailey/2011/12/14/annotated-source-code-as-documentation-with-docco/)) which can be found here: [Underscore](http://underscorejs.org/docs/underscore.html), [Backbone](http://backbonejs.org/docs/backbone.html), [Marionette](http://derickbailey.github.io/backbone.marionette/docs/backbone.marionette.html).

Use libraries, don't let frameworks use you!
