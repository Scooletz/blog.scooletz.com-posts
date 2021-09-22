---
layout: post
title: "Durable Functions' events made twice as fast"
date: 2021-09-27 11:00
author: scooletz
permalink: /2021/09/27/durable-functions-events-made-twice-as-fast
image: /img/2020/10/azure-functions.png
categories: ["azure", "AzureFunctions", "DurableFunctions", "async"]
tags: ["azure", "AzureFunctions", "DurableFunctions", "async"]
whitebackgroundimage: true
---

_Durable Functions_ are an implementation of the Durable Task framework within the serverless environment of _Azure Functions_. They allow you to leverage the power of the orchestration and write long-running, process oriented functions capable of dealing with failures, timeouts and all the other _interesting problems_ of distributed systems. In this post I discuss a [PR that I made](https://github.com/Azure/azure-functions-durable-extension/pull/1945) that lowers both, the CPU and the memory usage.

_A personal note: It's been a while since I wrote something. It was a turbulent time. When going back, sometimes it's good to walk the paved road, the one that you know. This was probably the most important driver of this work._

### Durable Functions

Instead of repeating the mantra of `fan-out` or `chaining`, which are straight-forward, let me mention that Durable Functions are capable of running a really complex flows based on asynchronous primitives. You can see in them regular async-aware constructs like `Task`, `CancellationTokenSource` and all the functions aggregating tasks like `Task.WhenAny` and `Task.WhenAll`. Additionally, they are capable of calling other activities that are responsible for performing actual actions. It's worth to mention, that performing any actual computation or IO should take place outside of a durable function. The durable function is meant only to orchestrate! If you're interested how the check is performed, whether the IO happened in there, take a look at [the source code of Durable Task](https://github.com/Azure/durabletask/blob/d89ca586ff9ab4bb7443562beeb3f501783130f7/src/DurableTask.Core/OrchestrationContext.cs#L28-L33) and follow to [the snippet from the Durable Functions](https://github.com/Azure/azure-functions-durable-extension/blob/a3070803a7c20e7acf6b49d9457a7d086b17040e/src/WebJobs.Extensions.DurableTask/ContextImplementations/DurableOrchestrationContext.cs#L1119-L1123).

### The discovery

When going through the implementation of Durable Functions, I was looking at how the code is structured? What patterns are used in there? How is it coupled to the Durable Task? One of the problems were target frameworks, which unfortunately include `.netstardard` and a good old `.NET Framework`. No `.NET Core` nor `.NET5` in there! This meant that all the performance opportunities that one can bring to a modernized project (like: `Spanification`) weren't available.

What I did is that I spent a healthy amount of time sniffing around [DurableOrchestrationContext](https://github.com/Azure/azure-functions-durable-extension/blob/dev/src/WebJobs.Extensions.DurableTask/ContextImplementations/DurableOrchestrationContext.cs) which is created for every execution of the orchestration. This looked like a good point to search for some potential optimizations. The serialization stuff wasn't available. There's a lot of coupling in there and a great potential for breaking the backward compatibility... There was the thing that interested me though...

### Non-generic over generic

The thing that interested me were [these lines](https://github.com/Azure/azure-functions-durable-extension/blob/fc58a62aa0b82da32eded1cddc1b1f2f2b97364a/src/WebJobs.Extensions.DurableTask/ContextImplementations/DurableOrchestrationContext.cs#L878-L914):

```csharp
object tcs = taskCompletionSources.Pop();
Type tcsType = tcs.GetType();
Type genericTypeArgument = tcsType.GetGenericArguments()[0];

// some code removed

object deserialized = this.converter.Deserialize(input, genericTypeArgument);

// some code removed

MethodInfo trySetResult = tcsType.GetMethod("TrySetResult");
trySetResult.Invoke(tcs, new[] { deserialized });
```

They were showing a common issue of calling some generic components from non-generic ones. As you can see the execution is as follows:

1. an object representing a generic `TaskCompletionSource<T>` is retrieved
1. it's type is inspected and used
1. a generic method is called with a non generic parameter

This **non-generic over generic** can be solved in many ways. One of them is implementing a non-generic interface by a generic component and make it accessible to a generic-angostic code. This was the first part of the solution that I provided

```csharp
private class EventTaskCompletionSource<T> : TaskCompletionSource<T>, 
  IEventTaskCompletionSource
{
  public Type EventType => typeof(T);

  void IEventTaskCompletionSource.TrySetResult(object result) => 
    this.TrySetResult((T)result);
}

private interface IEventTaskCompletionSource
{
  Type EventType { get; }

  void TrySetResult(object result);
}
```

It addressed all the issues and allowed to set a result in a generic-agnostic way without the usage of reflection. Additionally, it allowed to  probe for the type. Again, without the usage of the mighty reflection.

### Stack me not

The second change, which was much more subtle, was related to the data structure used to keep the events. Previously it was a `Stack<>`. It was used to stack up `TaskCompletionSource<>` instances. But now, when there was a custom one and the methods for raising and waiting were the only callers of the stack, the stack could be replaced by a simple `.Next` property within the `EventTaskCompletionSource<T>`. There was no need to have an additional data structure allocated for a simple pop and a simple push! This change amended the data structure and introduced the `.Next`.

```csharp
private class EventTaskCompletionSource<T> : TaskCompletionSource<T>, 
  IEventTaskCompletionSource
{
  public Type EventType => typeof(T);

  IEventTaskCompletionSource Next { get; set; }

  void IEventTaskCompletionSource.TrySetResult(object result) => 
    this.TrySetResult((T)result);
}

private interface IEventTaskCompletionSource
{
  Type EventType { get; }

  IEventTaskCompletionSource Next { get; set; }

  void TrySetResult(object result);
}
```

As you can see, it uses the non-generic version to make it callable from the non-generic methods of the `DurableOrchestrationContext`.

### Benchmarks

As always, any guesses in the performance related work should be supported by a valid set of benchmarks. Otherwise, and this is a separate history for a PR that I didn't issue, they might not be worth it. In this case, the gains were really promising.

|             Method |       Mean |    Error |   StdDev |  Gen 0 | Allocated |
|------------------- |-----------:|---------:|---------:|-------:|----------:|
|           OneEvent |   653.7 ns | 12.75 ns | 11.30 ns | 0.1345 |     424 B |
| **OneEvent (after)**| 256.4 ns |  5.10 ns |  8.52 ns | 0.0558 |     176 B |
| TwoDifferentEvents | 1,398.9 ns | 27.22 ns | 26.73 ns | 0.3223 |   1,016 B |
| **TwoDifferentEvents (after)** | 628.1 ns | 12.35 ns | 15.62 ns | 0.1650 |     520 B |
|      TwoSameEvents | 1,357.4 ns | 26.82 ns | 41.75 ns | 0.2766 |     872 B |
|      **TwoSameEvents (after)** | 641.7 ns | 12.37 ns | 11.57 ns | 0.1631 |     512 B |

As you can see, they save a few hundred bytes per event raised as well as some precious CPU time. These numbers might not look big. Two things are worth to mention though. The first, is that I run the benchmark over already heavily optimized .NET 5. The second, these things work in millions, if not billions, of orchestrations. The saving is cumulative!

### Summary

I hope you enjoyed the reading. I wish you good guesses, even better performance improvements and massive gains reported by your benchmarks.
