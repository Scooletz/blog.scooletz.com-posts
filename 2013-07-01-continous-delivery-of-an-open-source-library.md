---
layout: post
title: "Continous delivery of an open source library"
date: 2013-07-01 10:00
author: scooletz
permalink: /2013/07/01/continous-delivery-of-an-open-source-library/
nocomments: true
categories: ["Continous Integration"]
tags: ["CI", "Continuous delivery", "NuGet", "TeamCity"]
imported: true
---

In [the recent post](http://blog.scooletz.com/2013/06/27/evolving-your-branching-strategy) I've dealt with a basic setup of git branching protecting an open source library author from a headache of not-so-production-ready master branch of a repository. Having two main branches setup (master & dev) one can think about introducing continuous delivery. How about automatic publishing any set of commits getting into production branch? TeamCity with its git support made it trivial.

The very first build configuration has been made for dev branch. The target branch was dev branch. It consists of only two steps:

1. Build - done with MsBuild
1. NUnit - running all the tests

The second was a bit longer, based on the master:

1. Build - done with MsBuild
1. NUnit - running all the tests
1. NuGet pack - preparing a NuGet package with mixed NuSpec + csproj of the main library
1. NUnit publish - pushing the prepared NuGet to the [NuGet gallery](http://nuget.org/).

As the master branch is considered as production ready, in my opinion, there's nothing wrong with creating and uploading NuGet package for each set of commits, which goes through the tests. The very last case is versioning. For now (KISS), it's based on the master branch build number preppended and appended with 0. This generates 0.1.0, 0.2.0, 0.3.0, etc.
