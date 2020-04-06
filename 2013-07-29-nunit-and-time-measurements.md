---
layout: post
title: "NUnit and time measurements"
date: 2013-07-29 10:00
author: scooletz
permalink: /2013/07/29/nunit-and-time-measurements/
nocomments: true
categories: ["Uncategorized"]
tags: []
imported: true
---

It's common, that some part of your NUnit tests are tests, that should log their execution time. One of the ways of providing such a behavior is to provide custom *SetUp* and *TearDown* methods whether in your fixture or in a base test fixture. I find it disturbing, as a simple SetUp can be bloated with plenty of concerns.

Another way of providing it is using not a well-known interface of [ITestAction](http://nunit.org/index.php?p=actionAttributes&r=2.6.2 "ITestAction"). It allows to execute code before and after test in an AOP way. Of course one can argue that a simple method accepting action which execution will be measured is a better option, but I prefer coding in a declarative way and using a simple attribute visible in the signature of your method seems much more suited for this kind of behavior.
Take a look at the gist below and use it in you tests!

[gist]https://gist.github.com/Scooletz/6099052[/gist]
