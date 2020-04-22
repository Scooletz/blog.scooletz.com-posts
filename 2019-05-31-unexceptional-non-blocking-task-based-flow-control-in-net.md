---
layout: post
title: "Unexceptional non-blocking task-based flow control in .NET"
date: 2019-05-31 08:55
author: scooletz
permalink: /2019/05/31/unexceptional-non-blocking-task-based-flow-control-in-net/
image: /img/2019/06/flow.png
categories: ["async", "dotnet", "Azure", "Azure Functions", "serverless"]
tags: ["async", "dotnet", "Azure", "Azure Functions", "serverless"]
whitebackgroundimage: true
---

*This article shows in a simple way the foundation of libraries like **DurableTask** and its descendant **Azure Durable Functions.** It starts with the history of exception based flow control and ends by showing how Task-based API can be used to provide a similar interruptible flow with no abusive exception usage.*

### Dark ages of exception-driven programming

Imagine the following scenario. You provide a library or a service that enables executing users' code. Think of it as a hosted execution with some dependency. The usual interaction would be following:

1. You want to provide some kind of a storage API, that is passed to the consumer
1. The consumer of your API, provides an implementation that accepts your components.

This setup could look like the one below

```csharp
interface IExecutable
{
  public void Run(IContext context);
}

class YourExecution : IExecutable
{
  public void Run(IContext context)
  {
    context.CallService("some data");
    // and more
  }
}
```

If for any reason, like breaching the number of messages being sent per second, you'd like to stop the execution, the only way to do this, would be to throw an exception from *SendMessage* hoping, that it won't be caught with *try-catch*. It was doable, but still, on the higher level of execution it would result in an exception driven approach.

### Freeze

Another approach, that could be used with the synchronous API would be usage of a synchronization primitive like [Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore). This would enable to stop the execution from happening. This would also result in having a thread that does nothing and is simply waiting for better times to come.

### Task-based control flow

A much better way to control the flow, especially of the async enabled code, is understanding that the continuation of the code after *await* is captured to be executed but does not have to be executed at all. Everything depends on the status of the *Task* that is returned from the call. For example, in the snippet below, the intent of calling B will be captured and added as a continuation to A.

```csharp
await A();

await B();
```

This means, that if we had an ability to return a task that is arbitrarily either *Completed* or never completed, we could control the flow, either allowing its execution or dropping it completely.

To satisfy the positive path, *Task.CompletedTask* could be used. For the never completed, it could be *Task.Delay(Timeout.Infinite)* could be used. Let's try to implement it

```csharp
class Context : IContext
{
  public async Task CallService(string data)
  {
    if (shouldExecute)
    {
        // maybe do something
        return Task.Completed;
    }
    else
    {
        return Task.Delay(Timeout.Infinite);
    }
  }
}
```

You could ask a question, how the framework for executing the new async implementation of *IExecutable* could know what path was executed. And how to await on it? The solution can be provided by the following context

```csharp
class Context : IContext
{
  public Task Ended => tcs.Task;

  readonly TaskCompletionSource tcs =
   new TaskCompletionSource(
   TaskCreationOptions.RunContinuationsAsynchronously);

  public Task CallService(string data)
  {
    if (shouldExecute)
    {
        // maybe do something
        return Task.Completed;
    }
    else
    {
        // inform the host
        EndCurrentExecution();

        return Task.Delay(Timeout.Infinite);
    }
  }

  void EndCurrentExecution()
  {
    tcs.SetResult(this);
  }
}

```

With this approach, the host of the execution should wait on one of two events:

1. The execution is ended normally
1. The context has its *Ended* completed, marking that the execution was interrupted because of some reason (in Azure Durable Functions, it would be a call to the activity function).

Let's see the final code hosting the execution

```csharp
public static async Task Execute (IAsyncExecutable e)
{
  var ctx = new Context();

  var completed = await Task.WhenAny(
     e.RunAsync(context), ctx.Ended);

  if (completed == ctx.Ended)
  {
    // the execution ended by context
  }
  else
  {
    // the execution run to its end
  }
}
```

A similar flow, based on understanding what path of the code was executed, can be used to build orchestrations similar to **DurableTask** and **Azure Durable Functions**. One could argue that it's stretching async-await a bit too far. My take on this is, that it leverages what's already in there: a continuation being captured by the compiler, that transforms async-await into a regular state machine.

### Summary

Controlling flow is a complex concept, especially when you consider code that you don't own. The past way of doing it with exceptions is gone. It's about time to leverage the already chopped async-await flow and know which path was executed.

Happy async-await-ing :-)
