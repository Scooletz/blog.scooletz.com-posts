---
layout: post
title: "Task, Async Await, ValueTask, IValueTaskSource and how to keep your sanity in modern .NET world"
date: 2018-05-14 08:55
author: scooletz
permalink: /2018/05/14/task-async-await-valuetask-ivaluetasksource-and-how-to-keep-your-sanity-in-modern-net-world/
image: /img/2018/05/scan.png
categories: ["async", "dotnet"]
tags: ["async", "dotnet"]
whitebackgroundimage: true
---

### TL;DR

C# and .NET went through a lot of changes in recent years. First, came Task Parallel Library. Later, async-await made everyone adding `Async` suffix to their methods and putting a lot of *awaits*. Recently, we've been told that `Task` might be too heavy for synchronous cases (whatever it means) and `ValueTask` might be a good alternative. The last but not least, .NET Core 2.1 introduces a mind blowing construct of `IValueTaskSource` that enables allocate even less for the asynchronous path...

Complex enough? If you're starting your journey or started it some time ago but you didn't spend countless hours on following all these performance related tweaks, this article is for you. I'll build up your knowledge assuming just some basics. If initial parts are too easy, don't hesitate to jump a few paragraphs. This is not a whodunit. Still, I'll try to build up some tension ;-)

### Struct vs class, stack vs heap

We need to cover some basics first. In the managed .NET world, there are two memory regions you should care about: stack and heap. The heap is the managed memory where all objects are allocated (whenever you call `new YourClass ()` ). The important aspect is that it's not related or connected to any thread. One thread can allocate an object and pass it to another thread. The object will be on the heap as long as it's not collected by Garbage Collector.

The stack works in a different way. It's assigned to a specific thread (every thread has a limited size of its stack). The stack has frames, and can be used to allocate some memory, for instance for `struct` values that you create and assign to a variable during a call, for example via calling `Guid.NewGuid()`

```csharp
// the memory, for the value stored "in" the id,
// is on the stack
var id = Guid.NewGuid();
```

Effectively, you need to remember, that the heap allocated memory can be used to transfer some data between threads. The stack, because of it's assignment to a specific thread, can't be used for this purpose. On the other hand, if you want to allocate small amount of memory, the stack will be a bit faster as it's local to the thread and cannot fall into the slow path of starting the Garbage Collector, etc.

We know a few things about memory, and which kind of memory can be used for passing captured data across threads. It's time to take a look at what Task is.

### Tasks

Let's assume that you want to run a specific piece of code, a method. Before tasks, majority of us were using threads, either creating a `new Thread` manually, or using a fancy wrapper called `BackgroundWorker`. There were some issues with this approach. For example, if you were using a CPU with 4 cores (no hyper-threading) and you just started 5th thread, at least 2 of them were being executed on the same core. This wasn't efficient and led to a context switching.

> The **context switching** is a process of unloading a thread state from the memory, remembering it, and reloading another thread. The more threads, the more switching, the more taxation on the work that should be done.

Imagine, that instead of creating a thread for each of the tasks you want to run, you'd define a method that should be run. Now, the whole scheduling, the whole thread management would be taken care of. That's effectively the task model. You pass a delegate that should be run and *voila*, it will just work. Underneath, there are few interesting mechanisms like work stealing queues (to load balance the workload across the pool), but we'll keep focus on the task itself.

### Async await

Who doesn't like async-await? In the past, we had `void`, now we need to `Task`-ify our methods. In the past, there was no keyword that you need to put before every method call, now we have one, it's `await` (ofc if method is async itself). You may cry, wine, but async-await is here and it expresses, as a friend of mine [Daniel Marbach](https://twitter.com/danielmarbach) says, an opportunity for optimization. The code after await may be run on different thread, but if the data are there, or simply operation requires no asynchrony, it can be just continued on the same thread.

```csharp
var result = await GetDataAsync();
// we might be running the rest
// of this code on different thread
```

You should make no assumption about the thread that the continuation will be run on.

> The **continuation** is the code that follows an await statement. C# compiler transforms it into a special structure that captures the data and methods being called. Effectively, your asynchronous code consists of continuations.

### Which thread is it anyway

We know, that we don't know which thread will continue to run the rest of the code, after the `await` statement. If this was the same thread, we could just use the thread's stack to pass all the parameters (we know it from the first paragraph). But if this is another thread, we know that we can pass data only via object that is allocated on the managed heap. How can this be done?

The trickery is hidden in the C# compiler. What it does, is the following. For an `await` it will generate a state machine, a simple struct that will capture all the data needed to run the continuation (the code after *await*). An example async state machine looks like this:

```csharp
[CompilerGenerated]
[StructLayout(LayoutKind.Auto)]
private struct \u003CAsyncModeCopyToAsync\u003Ed__135
: IAsyncStateMachine
{
// fields with state omitted
}
```

As you can see, this is a `struct`, so when it's instantiated, it will be put on the stack. We know already, that it is cheap and fast. If the continuation is run on another thread, we cannot use the struct as we cannot pass the stack from one thread to another. The only way to do it, it's to put in on the heap.

As the state structure implements the interface, we can use the casting to the interface to box it and put in on the heap. With this boxed value, it can be passed to another thread.

> The **boxing** operation effectively, creates an object (on the heap) representing a value type. Boxing happens whenever a struct is cast either to `object`, or any other interface. There's an explicit `OpCode` box in .NET IL

### Why is ValueTask

There are cases where you don't want to write an async method using the C# compiler trickery. Sometimes you want to just return a value, but the method has the asynchronous signature, returning `Task`. Whenever this happens, you can use *Task.FromResult* to create the task artificially. Let's see the following example.

```csharp
public Task<int> GetIdAsync()
{
    return Task.FromResult (5);
}
```

This code is simple, does not create the whole async state machine magic but has one problem. It allocates as Task is class. Of course for the majority of the cases this won't be a problem. If you want to write a really fast code though, you might consider using an alternative called `ValueTask`, that is a struct. For the cases, where the implementation of a method might be synchronous, you could consider this as a better signature.

```csharp
public ValueTask<int> GetIdAsync()
{
    return new ValueTask(5);
}
```

This is the synchronous path. No IO, no network is called. What about these cases? What about the cases where you'd like to cover both: a fast non-allocating synchronous path and an asynchronous path but without allocating the Task. Is it possible?

### IValueTaskSource

In .NET prior .NET Core 2.1 this was not possible. `ValueTask` could be either created from the value (the synchronous path) or the task, that unfortunately allocates memory as it's a class. In .NET Core 2.1 there's a third option called `IValueTaskSource`.

Let's review what we had before core 2.1

```csharp
public readonly struct ValueTask<TResult>
{
    public ValueTask(Task task)
    public ValueTask(TResult result)
}
```

.NET Core 2.1 adds one more constructor for the ValueTask

```csharp
public readonly struct ValueTask<TResult>
{
    public ValueTask(IValueTaskSource<TResult> source, short token)
}
```

As you can see, this constructor requires no Task. It depends on an interface, that you can implement. Additionally, you can pool the objects implementing this interface, and return them to the pool as soon as the ValueTask is awaited. What is the `token` parameter? It's a value that ensures that the ValueTask won't be used after your `IValueTaskSource` is returned to the pool. Effectively, this token value, is passed to every method of your implementation of IValueTaskSource which enables you to check, if the caller does not abuse your ValueTask

```csharp
public interface IValueTaskSource<TResult>
{
    ValueTaskSourceStatus GetStatus(short token);
    void OnCompleted(Action continuation, object state, short token, ValueTaskSourceOnCompletedFlags flags);
    TResult GetResult(short token);
}
```

The `IValueTaskSource` approach, was used to modify and speed up implementation of the Socket class in .NET Core 2.1. The approach that was taken was following. A single socket can issue only a single send and a single receive at the same time (a call to the socket object). With this, only two objects implementing *IValueTaskSource* will be needed per socket, to provide same functionality, without any allocations. If you're interested into it, please take a look at the [implementation](https://github.com/dotnet/corefx/blob/master/src/System.Net.Sockets/src/System/Net/Sockets/Socket.Tasks.cs#L808-L1097).

### Why should I care

As you can see, in the last .NET Core 2.1 both, the synchronous and the asynchronous path of your async-await code can be executed without additional allocations. Actually, the socket code has already been ported to this approach bringing massive performance gains.

The good part is that you probably won't need to do in on your own. If you stick to regular async-await, soon, you'll see some methods will change their signature from `Task` to `ValueTask`. This change won't impact your code though. You'll need just to recompile your code and it will allocate less.

### Summary

It's interesting how long it took to get the asynchrony right. On the other hand, the recent changes in .NET Core 2.1 pushed it even further, removing unneeded allocations and leaving a lot of space for performance oriented software. With this, .NET Core moves even closer to the system programming space. The best part is that we'll all benefit from it.

### References

1. [ValueTask](https://github.com/dotnet/coreclr/blob/85374ceaed177f71472cc4c23c69daf7402e5048/src/System.Private.CoreLib/shared/System/Threading/Tasks/ValueTask.cs#L438)
1. [.NET Core 2.1 Performance Gains](https://blogs.msdn.microsoft.com/dotnet/2018/04/18/performance-improvements-in-net-core-2-1/)
1. [Awesome stack overflow question](https://stackoverflow.com/questions/43000520/why-would-one-use-taskt-over-valuetaskt-in-c)
