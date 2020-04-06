---
layout: post
title: "AzureDirectory - code review"
date: 2014-03-14 10:00
author: scooletz
permalink: /2014/03/14/azuredirectory-code-review/
nocomments: true
categories: ["Code review"]
tags: ["azure", "AzureDirectory", "Lucene"]
imported: true
---

The project AzureDirectory provides an Azure implementation of the Lucene.NET abstraction - *Directory*. It targets in providing ability to store Lucene index in the Azure storage services. The code can be found in here [AzureDirectory ](https://azuredirectory.codeplex.com/ "AzureDirectory @ codeplex"). The packages can be found on nuget [here](https://www.nuget.org/packages/Lucene.Net.Store.Azure/). There aren't marked as prereleases.

The solutions consists of two projects:

1. AzureDirectory
1. TestApp

The first provides implementation of the Lucene abstractions. The are a few classes, only needed for the feature implementation (implementations of Lucene abstractions). Additionally some utils class are introduced.
The code is structured with regions, which I personally dislike. Names of regions like: CTORS, internal methods, DIRECTORY METHODS shows the way the code is molded, with no classes holding common wrapped in region functionality. The lengthy methods and ctors are another disadvantage of this code base.
The spacing, using directives, fields that may be *readonly* are messy. Something which may be cleared with a ReSharper *Clean Code* is left for a reader to deal with.

You can find in there usages of Lucene obsolete API (like *IndexInput.Close* in disposal), as well as informative comments like:

> // sometimes we get access denied on the 2nd stream...but not always. I haven't tracked it down yet
> // but this covers our tail until I do

It's good and informative for author but leaving the project in an immature state.

The second project is not a test project but a sample app using the lib. No tests at all.

Summing up, after consideration, I wouldn't use this implementation for my production Azure app. The code is badly composed, with no tests and left with comments pointing at situations where authors are aware of the unsolved-yet problem.
