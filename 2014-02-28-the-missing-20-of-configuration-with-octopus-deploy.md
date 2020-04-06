---
layout: post
title: "The missing 20% of configuration with Octopus Deploy"
date: 2014-02-28 10:00
author: scooletz
permalink: /2014/02/28/the-missing-20-of-configuration-with-octopus-deploy/
nocomments: true
categories: ["Continous delivery"]
tags: ["Continous Integration", "Octopus Deploy"]
imported: true
---

Recently I've been evaluating [Octopus Deploy](http://octopusdeploy.com/). I wanted to learn more about a platform which is quite unique for .NET environment. After getting through the features I wrote following tweet:

> The [@OctopusDeploy](https://twitter.com/OctopusDeploy) feature of scoping variables to all the dimensions: environment x role x machine x step, should satisfy 80% of cfg needs.
> — Scooletz (@Scooletz) [February 25, 2014](https://twitter.com/Scooletz/statuses/438412504889569281)

What's about the missing 20%?
The missing part is the configuration considered as an artifact. There are many projects when changing you code makes you change your configuration. Adding some values is ok, much more important are operations like swapping whole sections, etc. This kind of changes are frequently connected with the given code change, hence they are changed in the very same commit and pushed to VCS. Then, your configuration is no longer a simple table with selection of dimensions where the given value should be applied. The additional skipped dimension is time measured in the only unit VCS is aware of - commits. It's like "from now on I need this values".
What you can do is to use Octopus values to point to the right file for the given environment, that's for sure. The thing becomes a bit more tricky when for instance production config should not be leaked into the development repo, etc.
This leads to the fact, that your configuration is an artifact. In many cases it can be easily replaced by a table with 'environment' dimension but still, it is an artifact, now, unfortunately not stored in your repository.

The reasoning above is not meant to lead you astray. Octopus is a great deployment tool. As with every tool, use it wisely.
