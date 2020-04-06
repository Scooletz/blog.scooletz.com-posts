---
layout: post
title: "An AMD friendly JavaScript module pattern"
date: 2013-07-04 10:00
author: scooletz
permalink: /2013/07/04/an-amd-friendly-javascript-module-pattern/
nocomments: true
categories: ["Javascript"]
tags: ["AMD", "javascript", "js", "module"]
imported: true
---

Recently I've been doing a few things with http cookies. I went through the specification and I know *path-match* and *domain-match*, hell yeah! One of the results of this trip was a nice JS module pattern found in a jQuery plugin for [cookies](https://github.com/carhartl/jquery-cookie "jQuery cookie"). Let's take a look!

[gist]http://gist.github.com/Scooletz/5853696[/gist]

In the line one to nine, a module method is created. If an AMD is found, then the passed *factory* function is used in the *define*. Otherwise a standard module pattern, with calling a function with a dependency passing is used. In lines 9-11 the real factory function is provided (that would be a whole standard module). The defined function is passed as a factory to the function defined above.

You can ship this code with AMD or without and it will work in both scenarios. What a pleasant way of providing library aware of its dependency resolving environment!
