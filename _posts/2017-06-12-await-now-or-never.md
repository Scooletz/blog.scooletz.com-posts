---
layout: post
title: "Await Now or Never"
date: 2017-06-12 08:55
author: scooletz
permalink: /2017/06/12/await-now-or-never/
nocomments: true
image: /img/2017/06/stocksnap_ceayzayehb.jpg
categories: ["C#"]
tags: ["async", "orchestration", "saga"]
imported: true
---

### Intro

This post is a continuation of [implementing a custom scheduler for your orchestrations](https://blog.scooletz.com/2017/06/08/implementing-a-scheduler-for-your-orchestrations/). We saw that the Delay operation is either completed or results in a never ending task, that nobody ever completes. Could we make it easier and provide a better way for delaying operation?

### Complete or not, here I come

The completed Delay operation was realized by

```csharp

Task.CompletedTask

```

This is a static readonly instance of a task that is already completed. If you need to return a completed task, because the operation of your asynchronous method was done synchronously, this is the best way you can do it.

For cases where we don't want continuations to be run, we used:

```csharp

new TaskCompletionSource<object>().Task

```

which of course allocates both, the TaskCompletionSource instance and the underlying Task object. It's not that much, but maybe, as there are only two states of the continuation: now or never, we could provide a smaller tool for this, that does not allocate.

### Now OR Never

You probably know, that you might create your custom awaitable objects, that you don't need to await on Tasks only. Let's take a look at the following class

```csharp

public sealed class NowOrNever : ICriticalNotifyCompletion
{
  public static readonly NowOrNever Never = new NowOrNever(false);
  public static readonly NowOrNever Now = new NowOrNever(true);

  NowOrNever(bool isCompleted)
  {
    IsCompleted = isCompleted;
  }

  public NowOrNever GetAwaiter()
  {
    return this;
  }

  public void GetResult() { }

  public bool IsCompleted { get; }

  public void OnCompleted(Action continuation) { }

  public void UnsafeOnCompleted(Action continuation) { }
}

```

This class is awaitable, as it provides three elements:

1. IsCompleted - for checking whether it was finished (fast enough or synchronously to do not build whole machinery for an asynchronous dispatch)
1. GetAwaiter - to obtain the awaiter that is used to create the asynchronous flow
1. GetResult

Knowing what are these parts for, let's take a look at different values provided by NowOrNever static fields
<table>
<tbody>
<tr>
<td>**NowOrNever**</td>
<td>**IsCompleted**</td>
<td>**OnCompleted/ UnsafeOnCompleted**</td>
</tr>
<tr>
<td>Now</td>
<td>true</td>
<td>no action</td>
</tr>
<tr>
<td>Never</td>
<td>false</td>
<td>no action</td>
</tr>
</tbody>
</table>


As you can see, the completion is never called at all. For the **Never** case, that's what we meant. What about **Now**? Just take a look above. Whenever *IsCompleted* is true, no continuations are attached, and the code is executed synchronously. We don't need to preserve continuations as there are none.

### Summary

Writing a custom awaitable class is not a day-to-day activity. Sometimes there are cases where this could have its limited benefit. In this NowOrNever case, this allowed to skip the allocation of a single task, although, yes, the created async state machine takes probably much more that a single task instance.
