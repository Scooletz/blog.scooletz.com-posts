---
layout: post
title: ".NET 5 and pooling for ValueTasks"
date: 2020-06-01 11:00
author: scooletz
permalink: /2020/06/01/pooling-for-value-tasks-in-net5
image: /img/2020/06/pooling-for-value-tasks.png
categories: ["dotnet", "performance", "async"]
tags: ["dotnet", "performance", "async"]
whitebackgroundimage: true
---

.NET 5 is around the corner and it will bring a lot of goodness to the land of performance. One of the interesting aspects that I visited when working on [Async Expert online course](https://asyncexpert.com) is the aspect of pooling boxes hidden behind `ValueTask` returning operations. The reasoning behind it can be found in the awesome article [Async ValueTask Pooling in .NET 5](https://devblogs.microsoft.com/dotnet/async-valuetask-pooling-in-net-5/) authored by Stephen Toub. This post focuses on diving deep in the codebase and seeing how it's implemented behind it.

### Enabling pooling

To enable pooling, the `DOTNET_SYSTEM_THREADING_POOLASYNCVALUETASKS` environment variable needs to be set either to `true` or `1`. Then the state machine box objects that implement the `IValueTaskSource` and `IValueTaskSource<TResult>` interfaces will be pooled and reused for async methods with `ValueTask` or async `ValueTask<TResult>`.

Another setting that you can use to tweak the behavior is `DOTNET_SYSTEM_THREADING_POOLASYNCVALUETASKSLIMIT` that sets the size of the pool. By default it's equal to `Environment.ProcessorCount * 4`.

Both can be found in [AsyncTaskCache](https://github.com/dotnet/runtime/blob/4f9ae42d861fcb4be2fcd5d3d55d5f227d30e723/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncTaskCache.cs#L34-L43) class.

### Caching condition

Setting the flag mentioned above is responsible for setting the field `AsyncTaskCache.s_valueTaskPoolingEnabled`. This is the value that is used to determine whether the pooling should be used on not. You can navigate through [AsyncValueTaskMethodBuilderT.cs](https://github.com/dotnet/runtime/blob/4f9ae42d861fcb4be2fcd5d3d55d5f227d30e723/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncValueTaskMethodBuilderT.cs) and [AsyncValueTaskMethodBuilder.cs](https://github.com/dotnet/runtime/blob/4f9ae42d861fcb4be2fcd5d3d55d5f227d30e723/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncValueTaskMethodBuilder.cs) to see all the occurrences. They are all quite similar and follow this pattern:

```csharp
public void AwaitOnCompleted<TAwaiter, TStateMachine>(ref TAwaiter awaiter,
    ref TStateMachine stateMachine)
    where TAwaiter : INotifyCompletion
    where TStateMachine : IAsyncStateMachine
{
    if (AsyncTaskCache.s_valueTaskPoolingEnabled)
    {
        // the value task pool way
        AwaitOnCompleted(ref awaiter, ref stateMachine,
            ref Unsafe.As<object?, StateMachineBox?>(ref m_task));
    }
    else
    {
        // the regular task way
        AsyncTaskMethodBuilder<TResult>.AwaitOnCompleted(ref awaiter, ref stateMachine,
            ref Unsafe.As<object?, Task<TResult>?>(ref m_task));
    }
}

internal static void AwaitOnCompleted<TAwaiter, TStateMachine>(
    ref TAwaiter awaiter, ref TStateMachine stateMachine, ref StateMachineBox? box)
    where TAwaiter : INotifyCompletion
    where TStateMachine : IAsyncStateMachine
{
    try
    {
        // GetStateMachineBox - it looks interesting :)
        awaiter.OnCompleted(GetStateMachineBox(ref stateMachine, ref box).MoveNextAction);
    }
    catch (Exception e)
    {
        System.Threading.Tasks.Task.ThrowAsync(e, targetContext: null);
    }
}
```

Now we're really close to see how pool works. Can you notice the `GetStateMachineBox` call? This is the place where pooling happens.

### Getting StateMachineBox

The method [GetStateMachineBox](https://github.com/dotnet/runtime/blob/c9f917052fc091014ef0f311b6f79a812bbfbd38/src/libraries/System.Private.CoreLib/src/System/Runtime/CompilerServices/AsyncValueTaskMethodBuilderT.cs#L202-L267) is 70 lines long and it's pretty well documented. It tries to obtain the object responsible for keeping the state a.k.a `box`. If it fails, it means that the box needs to be obtained or created as it's shown in its last few lines.

```csharp
var box = StateMachineBox<TStateMachine>.GetOrCreateBox();
boxFieldRef = box;
box.StateMachine = stateMachine;
box.Context = currentContext;
```

What's behind the `GetOrCreateBox` and can we finally move to the pooling behavior?

### Get or Create Box

The implementation of `GetOrCreateBox` is simple and uses fast primitives (read it as: no `lock`) to either obtain the existing box or fallback to the allocation path. Let's take a look at the code with comments that I augmented to explain some primitives used below.

```csharp
internal static StateMachineBox<TStateMachine> GetOrCreateBox()
{
    // Try to acquire the lock to access the cache.  If there's any contention, don't use the cache.
    // This operation uses an atomic compare and exchange.
    // The operation replaces s_cacheLock with 1 only and only if the previous value was 0.
    // It returns the previous value, so it can be checked whether the "fast lock" was taken.
    if (Interlocked.CompareExchange(ref s_cacheLock, 1, 0) == 0)
    {
        // The exchange succeeded. We can check the cache.
        // If there are any instances cached, take one from the cache stack and use it.
        StateMachineBox<TStateMachine>? box = s_cache;

        // check if there's something in the cache
        if (!(box is null))
        {
            // The cache is a simple list. Get item and  move it to the _next.
            s_cache = box._next;
            box._next = null;
            s_cacheSize--;
            Debug.Assert(s_cacheSize >= 0, "Expected the cache size to be non-negative.");

            // Release the lock. It uses Volatile.Write that will be eventually seen by other threads.
            // We can do it, as we know that we are holding the lock.
            // The other threads will finally see the s_cacheLock == 0.
            // One of them will be able to CompareExchange it with 1.
            // Volatile.Write ensures happened-before semantics. If another thread observes s_cacheLock == 0,
            // it also will observe a proper s_cache value.
            Volatile.Write(ref s_cacheLock, 0);
            return box;
        }

        // No objects were cached.  We'll just create a new instance.
        Debug.Assert(s_cacheSize == 0, "Expected cache size to be 0.");

        // There's no box, but again, we need to release the lock here. See comments above
        Volatile.Write(ref s_cacheLock, 0);
    }

    // Couldn't quickly get a cached instance, so create a new instance.
    return new StateMachineBox<TStateMachine>();
}
```

### Summary

I hope this posts helps you in diving a bit deeper in what's hidden behind pooling/caching for `ValueTask` in .NET5. As you can see, the behavior is optional and enabling it should be followed by measurements and benchmarking proving, that in your case it does help and reduces allocations greatly.
