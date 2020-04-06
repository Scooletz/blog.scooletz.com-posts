---
layout: post
title: "Implementing a scheduler for your orchestrations"
date: 2017-06-08 08:55
author: scooletz
permalink: /2017/06/08/implementing-a-scheduler-for-your-orchestrations/
nocomments: true
image: /img/2017/06/stocksnap_8nuxgzom1g.jpg
categories: ["C#", "modelling"]
tags: ["async", "orchestration", "saga"]
imported: true
---

### TL;DR

We've already seen [here](http://blog.scooletz.com/2017/04/13/orchestrating-your-processes-with-durable-task) and [here](http://blog.scooletz.com/2017/06/01/orchestrating-processes-for-fun-and-profit) that with *async-await* one could easily sketch an orchestration/saga for any process that should be both, robust and resilient. It's time to take a look how a scheduler for such a process could be implemented.

### Delay with no Task

Usually, when we want to delay an action in an asynchronous flow, we use [Task.Delay](https://msdn.microsoft.com/en-us/library/hh139096(v=vs.110).aspx). This method schedules a continuation with the rest of our code, to be executed after the specified delay. The usage is as simple as:

```csharp

await Task.Delay(TimeSpan.FromSeconds(1.5));

```

This is fine, when we want to postpone an action for a few seconds, but what in case of processes that want to be frozen for days? How could you implement it?

First, let us rephrase the delay to a method that is provided by the base orchestration class (you can always have a base, can't you?).

```csharp

await this.Delay(TimeSpan.FromSeconds(1.5));

```

With this assumption, we can move forward and take a look at a possible *Delay* implementation.

### Delay for Orchestrations

The whole idea of this orchestration is based on snapshoting its changes as events and make them replayable. In other words, if a failure occurs, the orchestration process should be resurrected on another node with no changes in the flow. This makes implementation a bit trickier, but is needed for providing strong foundations for our processes. Let's take a look at Delay possible implementation.

```csharp

protected Task Delay(TimeSpan delay)
{
  var date = GetDateTimeUtcNow();
  var scheduleAt = date + delay;

  ScheduledAt existingDelay;
  if (TryPop(out existingDelay))
  {
    if (existingDelay.Value > context.DateTimeUtcNow())
    {
      EndCurrentExecution();
      return new TaskCompletionSource<object>().Task;
    }

    return Task.CompletedTask;
  }

  if (scheduleAt <= date)
  {
    return Task.CompletedTask;
  }

  Append(new ScheduledAt(scheduleAt));
  EndCurrentExecution();
  return new TaskCompletionSource<object>().Task;
}

```

The first line of this methods calls to *GetDateTimeUtcNow*. As you can imagine, this gets the UTC current date. It has one additional property though. Do you remember that we need to make this method possible to be executed multiple times with the same effect? This means, that the result of *GetDateTimeUtcNow* will be recorded and when we, for any reason like the process kill, enter the orchestration again, it will provide the same value. Effectively, it will be **now** from the first execution of it.

The next step is to calculate the date when the delay should end, where the next execution is ScheduledAt.

We TryPop a prerecorded event. If the orchestration was already active, it left a trace, an event in the history, that we can pop. If there's an entry, we compare it with the current *UtcDateTimeNow*. If the orchestrations should wait more we just mark it as one that requires ending execution. Next we return  *new TaskCompletionSource<object>().Task* which effectively is a never ending task. This means that any continuation attached by the caller of this method, either explicit or implicit using *await* won't be run!

If there was no event and the date that Delay is scheduled for some reason lower then current date, a completed task is returned. Otherwise, an event is added and the current execution is ended with the same pattern: by setting a notification about execution not proceeding any longer and returning a never completing task.

### Execution status

The caller responsibility is to gather information whether the orchestration ended or was scheduled for later execution. This is done by awaiting one of the tasks. Either the task of the orchestration itself or a task that is *EndCurrentExecution* sets the result.

```csharp

await Task.WhenAny(
orchestration.Execute(),
currentExecutionEndingTask);

```

### Summary

We saw how powerful can be asynchronous flows, especially when connected with optional calling of scheduled continuations. With a simple recording of events we were able to create an orchestration tooling that is easy to use by the end user (programmer), but still provides an interesting and powerful semantics of a time dependent process.
