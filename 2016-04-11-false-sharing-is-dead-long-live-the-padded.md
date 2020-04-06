---
layout: post
title: "False sharing is dead, long live the Padded"
date: 2016-04-11 10:00
author: scooletz
permalink: /2016/04/11/false-sharing-is-dead-long-live-the-padded/
categories: ["Optimization", "RampUp"]
tags: ["False sharing", "Fody", "RampUpNet", "threading"]
imported: true
---

False sharing is a common problem of multithreaded applications in .NET. If you allocate objects in/for different threads, they may land on the same cache line impacting the performance, limiting gains from scaling your app on a single machine. Unfortunately, because of the multithreaded nature of the [RampUp](https://github.com/Scooletz/RampUp) library it's been suffering from the same condition. I've decided to address by providing a tooling rather than going through the whole codebase and apply [LayoutKind.Explicit](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.layoutkind%28v=vs.110%29.aspx) with plenty of [FieldOffsets](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.fieldoffsetattribute%28v=vs.110%29.aspx)

### [Padded](https://github.com/Scooletz/Padded) is born

The easiest and the best way of addressing cross cutting concerns in your .NET apps I've found so far is [Fody](https://github.com/Fody/Fody). It's a post compiler/weaver based on the mighty Mono.Cecil library. The tool has a decent documentation, allowing one to create a even quite complex plugin in a few hours. Because of this advantages I've used it already in RampUp but wanted to have something, which can live on its own. That how [Padded](https://github.com/Scooletz/Padded) was born.

### Pad me please

Padded uses a very simple technique of adding a dozen of additional fields. According to the test cases provided, they are sufficient enough to provide enough of space to prohibit overlapping with another object in the same cache line. All you need is to:

1. install Padded in your project (here you can find [nuget](https://www.nuget.org/packages/Padded.Fody/)) in a project that requires padding
1. declare one attribute in your project:

```csharp
namespace Padded.Fody
{
public sealed class PaddedAttribute : Attribute { }
}
```

1. mark the classes that need padding with this attribute.

### Summary

Marking a class/struct with one attribute is much easier than dealing with its layout using .NET attributes, especially, as they were created not for this purpose. Using a custom, small tool to get the needed result is the way to go. That's how & why [Padded ](https://github.com/Scooletz/Padded)was provided.
