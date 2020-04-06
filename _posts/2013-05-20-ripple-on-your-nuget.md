---
layout: post
title: "Ripple on your NuGet"
date: 2013-05-20 10:00
author: scooletz
permalink: /2013/05/20/ripple-on-your-nuget/
nocomments: true
categories: ["Continous Integration"]
tags: ["CI", "Fubu", "MVC", "NuGet", "package management", "Ripple"]
imported: true
---

Recently I've been involved in a project which consists of many teams collaborating to deliver a big project. The application architecture gives this teams an opportunity to work with their code in a module scope. There's no one big Visual Studio solution, all modules are scoped in their own slns. Living in a world like this means, that your CI, package management and local deployment must be fast, effective and easy to use.
[Ripple](https://github.com/DarthFubuMVC/ripple) is a project from the FubuMVC family. It's an alternative NuGet client, so all the changes in behavior are made on the client side. It makes it a perfect fit for a scenario where one can mix up a local NuGet feeds for their internal packages and official ones. What about its advantages?
The very first difference is a versioning. Ripple extracts packages into non versioned folders. It means that your projects will no longer be changed, when a new version of a package arrives. It helps a lot, especially in environments with packages being frequently published.
Ripple makes a distinction between referenced packages. There are *float* and *fixed* packages. Float packages are these changing rapidly. It's meant that all your packages should be floating during development. The fixed packages are rock solid versions of libraries used by your system. They will not be updated, unless the *force* option are used.

The Ripple is still being developed. The most important thing it's that it is active and thriving. If you develop applications in .NET with a plenty of solutions which are meant to publish packages, then give Ripple a try. It's worth it.
