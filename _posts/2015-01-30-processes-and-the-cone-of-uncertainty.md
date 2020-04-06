---
layout: post
title: "Processes and The Cone of Uncertainty"
date: 2015-01-30 11:00
author: scooletz
permalink: /2015/01/30/processes-and-the-cone-of-uncertainty/
nocomments: true
categories: ["Continous delivery", "Continous Integration", "Personal"]
tags: ["cone of uncertainty", "management", "tooling"]
imported: true
---

The Cone of Uncertainty is a management phrase describing the fact, that we cannot foresee the future of a project with a constant probability. The longer period we want to plan for, the less probable it's going to be exact. That's the reasoning behind sprints in Agile, to keep them short enough to be in the narrow part of the cone. But it isn't only the planning! The shape of the cone can be modified by some good practices and reducing the manual labor. By modified I mean greatly narrowed.
The are plenty of processes and elements that can be introduced:

* continues builds
* proper db deployments
* tests
* continues deployments
* promotion of application versions between environments

Each of them improves some parts of the development process making it less manual and more repeatable. Additionally, as you introduce tooling for majority of these cases, the tools run with a similar speed so you can greatly lower the uncertainty of some aspects of development. Then again, if some of aspects are constant, only the real development will affect the cone and your team with a manager get what you wanted: a more predictable process and a smaller cone of uncertainty.
