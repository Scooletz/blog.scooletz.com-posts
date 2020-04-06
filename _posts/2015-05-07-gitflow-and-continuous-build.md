---
layout: post
title: "GitFlow and Continuous Build"
date: 2015-05-07 10:00
author: scooletz
permalink: /2015/05/07/gitflow-and-continuous-build/
nocomments: true
categories: ["Continous delivery", "Continous Integration", "Git"]
tags: ["Continuous delivery", "git", "git flow"]
imported: true
---

In [the recent post ](http://blog.scooletz.com/2015/05/04/gitflow-and-code-review/ "GitFlow and code reviews")I've described the idea how to ensure, that your feature-to-develop GitFlow merge commits are reviewed before being introduced to the develop branch. This preserves the quality of the develop branch, ensuring that it's truly *possibly deployable*. How one would like to build his/her repository and provide artifacts? Which commits and which branches should be built? These questions are answered below.

Let's start with the following observation. Whichever branch points at the given commit, if a proper modern build approach is used like [PSake build script](https://github.com/psake/psake) is used, the result is the same. The repository contains all the needed scripts to run the build, the output will be the same no matter which branch is selected as a source of the build (if two or more points at the same commit). After all the same commit is the same tree which results in the same build. This gives us a very powerful tool in ensuring even better quality of develop. One can easily setup [TeamCity using branch selector](https://confluence.jetbrains.com/display/TCD9/Working+with+Feature+Branches) to run the same build for all features:

<tt>+:refs/heads/feature/*</tt>

The build script creates artifacts, in my case NuGet packages, using the following versioning [major].[minor].[build_number]. The first two are the numbers stored in the repository. This requires that features are not long running (you don't want to have a long running from 1.1.1 to get to later than 2.1.2). The build number is the same for all the features. For now, I'm not considering case whether the artifacts should be published to some gallery or not.

The question is, should we build the development commits? For now, considering only the feature branches, the answer is no! All the commits in the develop are merge commits which have been already reviewed and build in the feature branches! That's why creating the merge commit in the feature is so powerful. You get the code reviewed, built and tested before it goes to the development! Again, the idea of postponing finishing a feature till it has its value acknowledged bring profit to the quality of the develop branch.
