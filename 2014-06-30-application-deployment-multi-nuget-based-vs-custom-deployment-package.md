---
layout: post
title: "Application deployment: multi nuget based vs custom deployment package"
date: 2014-06-30 11:00
author: scooletz
permalink: /2014/06/30/application-deployment-multi-nuget-based-vs-custom-deployment-package/
categories: ["Continous delivery"]
tags: ["deployment", "NuGet", "Octopus Deploy"]
imported: true
---

So far I encountered a few patterns for deploying application. Speaking of those based on nuget packages, I can easily distinguish two of them.

### Package with references

that's the first one. Frequently it's not a custom package. It's based on the main solution project (the application part) and is pushed to a feed with packages based on other solution projects. This leads towards design where packages are:

1. small
1. meant to be cross-app reusable
1. mirrors the solution and project dependencies
1. has references to other packages
1. needs a nuget feed to resolve other dependencies during the deploytment time

Unfortunately, packages of this kind **are not stable** build artifacts. One can easily change multiple apps by pushing to the NuGet feed libraries used by installed projects. Packages once build and deployed may be changed between publications on environments which greatly diminishes the meaning of deployment package. Iff one totally controls pushing to the feeds and provides staging for feeds, this may work, otherwise - can be considered error prone (one cannot tell if the package published once, can be republished in the higher environment).

### Self-sufficient package

which is the second one. This kind of package, prepared specially for deployment, consists of all items required by the given deployment, which provides packages that are:

1. bigger
1. targeted towards deployment
1. orthogonal to a solution organization
1. has no references to other packages
1. needs no nuget feed to resolve other dependencies during the deploytment time

This kind of artifacts, used by [Octopus deploy](https://octopusdeploy.com/) consists of snapshots of all dependencies in the given moment of build. Snapshots, by default immutable and stable, brings a self-sufficient packages, which can be simply extracted in a given environment. This for the price of declaring a custom [nuspec](http://docs.nuget.org/docs/reference/nuspec-reference), brings the repeatable deployment on all the environments and is a preferable way of doing deployments of mine. Even if you don't want to use Octopus Deploy for some reasons.
