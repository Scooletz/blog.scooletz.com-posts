---
layout: post
title: "Performance in .NET as commodity"
date: 2019-05-21 08:55
author: scooletz
permalink: /2019/05/21/performance-in-net-as-commodity/
image: /img/2019/05/a.png
categories: ["dotnet", "performance"]
tags: ["dotnet", "performance"]
whitebackgroundimage: true
---

*This post is a short summary of my thoughts related to all the awesomeness that is being introduced to the .NET landscape, especially in regards to .NET Core (see [Performance Improvements in .NET Core 3.0](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-core-3-0/) an example). It was somewhat inspired by [Inertia as described in Wardley's maps](https://blog.gardeviance.org/)*

### Commodity

What is a commodity?

> All goods that the market treats instances of the good as equivalent or nearly so with no regard to who produced them.

Let me bend this definition a little bit, as with your runtime environment for sure you care who delivers the package. You wouldn't install just a *dotnet.exe* without having it signed, would you? ;-) The point that I'm trying to extract from this definition is that the thing is becoming common. Like flour or milk. You can use a general flour and you'll be fine.

### Performance

I love performance. Not matter if we discuss a cloud environment or ASM, I really like CPUs spending time efficiently. After [BenchmarkDotNet](https://benchmarkdotnet.org) becoming a standard for performance oriented code and then, seeing that is being used all over the place in Dotnet Foundation repositories, I thought that it's a really interesting movement. The best part that can get it for free. What do I mean by *free* then?

### Free

The first part, the most meaningful one is that if you run your application on .NET Core 3.0 it will run faster! For Linux, it will even stop allocating that much when doing async-await. This is really cool as it requires just to run on the new runtime

The second meaning of free, is the free time, not spent on working on the performance. How is it different from the last point? Do you remember the time when ConcurrentQueue was not that fast? When it was worth it to implement your own MPSC queue as it was faster? Guess what. Now, you don't have to do it as you won't be able to squeeze much more (maybe one or two Interlocked invocations per operation). It's generally faster and better designed to let you use what's there.

### Are we done with performance

I'm far from saying that the performance work is done, and that by just smashing keys and using BCL all over the place, you'll get it right. I'm just saying that the threshold of getting it messed up really badly has been moved. It will be easier to get your applications generally performing better, allocating less and performing when needed.
