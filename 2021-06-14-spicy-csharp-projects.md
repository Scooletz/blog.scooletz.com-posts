---
layout: post
title: "Spicy C# projects"
date: 2021-06-14 11:00
author: scooletz
permalink: /2021/06/14/spicy-csharp-projects
image: /img/2021/06/csharp.png
categories: ["csharp", "dotnet"]
tags: ["csharp", "dotnet"]
whitebackgroundimage: true
---

When recently we published the whole [C&#35; 9 Professional course](https://procsharp9.com/?utm_source=blog.scooletz&utm_medium=link&utm_campaign=procsharp9), I had a moment to pause and to reflect on all the new and shiny libraries and tools that were recently released in .NET ecosystem. Tools that are either based on the new C&#35; features or use other extraordinary approaches to deliver amazing results. Let's dive into my strongly subjective selection of the awesome spicy-hot projects!

### Small Sharp

The first project that I want to cover is [SmallSharp](https://github.com/devlooped/SmallSharp). It can come in handy, when you want to present multiple demos or samples and you don't want to create a project for every single one. If you take into consideration the recent advancement in C&#35; providing top level statements, which unfortunately support only one top-level program in a project, this might be an obstacle. SmallSharp helps in addressing this.

> SmallSharp brings that very feature by automatically generating a launch profile for each .cs file at the project level so you can just select which top-level program should be the startup item (for compilation and launch/debug).

This means, that you can easily create simple top-level programs and the whole overhead for creating launch profiles will be take care for you. Now revisit all the demos, samples etc. that you needed to create. Think big with SmallSharp.

### Mapping Generator

The [Mapping Generator](https://www.mappinggenerator.net) is another tool that uses the code generation capabilities to lower the number of runtime issues and bringing them back to the design time. What it focuses on is mappings and their generation. With Mapping Generator you won't be hit by a runtime error and instead you'll be provided with a build time error. This movement, to moving from runtime to build time, without even considering AOT scenarios, was visible in multiple projects for a while now. I'm glad to see that the topic of mapping objects has found its solution provided with this project.

### Netherite

The last one is not related to the new C# features as much, but it's still spicy hot. The project I'm referring to is [Netherite](https://github.com/microsoft/durabletask-netherite) and is meant to deliver great capabilities for Durable Functions' users _who has an appetite for performance, scalability, and reliability_. The main thing that I want to focus on in `Netherite`, is its usage of [FASTER](https://github.com/microsoft/FASTER), a project born in Microsoft Research that appeared, was talked about and then it went silent. It aims at delivering two components, a log and a key value store. It's interesting to see it's reappearing. Its idea of a device, something that can be written to and read from, can be supported by blobs and it delivers a great performance that Netherite is based on.
