---
layout: post
title: "Pass the State"
date: 2025-10-08 06:00
author: scooletz
permalink: /2025/10/08/pass-the-state
image: /img/2025/pass-the-state.webp
categories: ["csharp", "dotnet"]
tags: ["csharp", "dotnet"]
whitebackgroundimage: true
twitter: false
---

You’re writing the most beautiful piece of code ever. It can be a snippet that is LINQ-based, or something else that requires you to pass a function. You add just what is needed…

```c#
// Your initial code
public IEnumerable<Item> GetOversized(IEnumerable<Item> items)
{
    return items.Where(item => item.Size > 5)
}
```

until you notice that parameters of the function are not enough and you need to get some parameters out of thin air. What do you do in such a case? It’s tempting to just add it as is

```c#
public IEnumerable<Item> GetOversized(IEnumerable<Item> items, int minOversize)
{
    return items.Where(item => item.Size > minOversize)
}
```

For snippets that are rarely executed, it will just work. What about the cases where it’s the hot path? What about cases where it touches concurrency? Or the ones that involve cancellation with `CancellationToken` and other non-trivial cases? Let’s dive into it\! But first, we need to understand what we can do from the C\# point of view.

# Lambda strikes back

C\# provides us with a powerful tool called lambda. It allows us to define an anonymous function that can be passed as a Func or Action. It’s actually the very same thing that we passed to `.Where` in the two examples below: 

```c#
item => item.Size > minOversize
```

You define params and then, after an arrow, you declare what this anonymous function will do. 

There’s a caveat there that is hidden from you. Such functions can “capture the scope”. Capturing the scope means that they might grab some variables and parameters and wrap them in an object. Let’s re-examine the second example with a pseudo code that will be generated:

```c#
public IEnumerable<Item> GetOversized(IEnumerable<Item> items, int minOversize)
{
    // minOversize comes from the parameter.
    // It will be captured and an object will be alloctated.
    // This will be turned into something like:
    var captured = new GeneratedClassForMinOversize (minOversize)
    return items.Where(item => captured.Predicate(item))
}
```

This might not seem to be a big issue, but if such cases pile up in a codepath that is frequently executed, you might be burnt by allocations.. You could ask now: is there anything that I could do to make it explicit? Yes, there’s such a thing.

C\# 9.0 introduced a notion of static anonymous functions. These are the functions that allow you to add static to their declaration. Static, in this case, means that it is forbidden to capture the scope.

```c#
public IEnumerable<Item> GetOversized(IEnumerable<Item> items, int minOversize)
{
    // This won’t compile. Static + minOversize usage contradict each other
    return items.Where(static (item) => item.Size > minOversize)
}
```

Now, you may ask if it is always possible to not capture the scope. Unfortunately, this is not the case. To use static anonymous functions, one of two conditions needs to be met:

1. The function requires only the parameters that are provided  
2. There is an overload that accepts the state passing as an additional parameter.

The first one is trivial \- you just work with what you have. The second is less trivial. Still, once you see the pattern, you'll follow it in any case that supports it. Let’s consider the following sample:

```c#
public IEnumerable<T> Query(Func<T, bool> predicate);
```

Clearly, if there’s some external state that is required for the predicate, it will require capturing it in the scope. What do we see in the following overload though:

```c#
public IEnumerable<T> Query<TContext>(Func<T, TContext, bool> predicate,
    TContext ctx);
```

There’s an additional parameter, called context, that you can pass to the function. The crux lies in the fact that it is passed to the predicate as well. If you have a well defined context, you can just pass it in as an additional parameter, assuming that the underlying implementation will use it wisely. What would such a predicate look like then?

```c#

repo.Query((item, ctx) => (item) => item.Size > ctx, minOverSize);
```

We could make it even better, by providing the explicit requirement of not capturing the state. All it takes is a single static modifier\!

```c#

repo.Query(
  // ctxMinOverSize is the parameter that will pass minOverSize to our lambda
  static (item, ctxMinOverSize) => (item) => item.Size > ctxMinOverSize,
  minOverSize);
```

Now we know that if we combine the static lambdas and a context parameter we can do a lot, without capturing the state (allocating). And we know what to search for\!

# Aggregate like there’s no tomorrow

Let’s start our exploration with some simple cases. The first is as explicit as one can get. It’s the aggregating function of an enumerable.

```c#

public static TAccumulate Aggregate<TSource,TAccumulate>(this  IEnumerable<TSource> source, TAccumulate seed, Func<TAccumulate,TSource,TAccumulate> func);
```

Let’s analyze it:

1. It’s an extension method, that runs over an enumerable  
2. It accepts the context in a form of the seed parameter  
3. It has a function that accepts the context and an item of the enumerable. 

The fact that it returns the value of the context type is a design for the aggregating function. It has nothing to do with the idea of the context passing. How could we use it then?

```c#

var sumOfEven = numbers
   .Aggregate(0, static (a, item) => a + (item % 2 == 0 ? 1 : 0));
```

As you can see above: we passed the context, we used the static lambda, which resulted in no context captured. Is that all though? Are there other meaningful examples?

# ConcurrentDictionary factory

The next example is much less forced than the aggregation. Again, it’s based on a method that accepts a delegate and some state. In this case, we're looking at ConcurrentDictionary, with one of overloads of AddOrUpdate:

```c#
TValue AddOrUpdate<TArg>(TKey key, 
    Func<TKey,TArg,TValue> addValueFactory, 
    Func<TKey,TValue,TArg,TValue> updateValueFactory, 
    TArg factoryArgument);
```

This function accepts not one delegate but two\! The more delegates the merrier\! How does it work then? The ConcurrentDictionary is a dictionary that can handle multiple actors/threads accessing it at the same time. One of the common scenarios is adding or updating a value. A poor example (there are better ways to do this) is using a concurrent dictionary as a counter with different keys in it. 

```c#
var counters = new ConcurrentDictionary<string, long> ();

// Not the best counters, just showing API
counters.AddOrUpdate("tag", 
    (_, v) => v, // addValueFactory, set to the initial
    (_, prev, update) => prev + update, // updateValueFactory, sum up
    1); // the initial factory argument
```

What would be the states of such a counter?

1. A counter does not exist for a specific key and requires creating it  
2. A counter exists for a specific key and requires updating it.

To handle the first case, we’d like to have the following:

1. A key  
2. A delegate to create the value that accepts the key and the context.  
3. A context as the delegate might require it

```c#
TValue AddOrUpdate<TArg>(TKey key,                   // the key
    Func<TKey,TArg,TValue> addValueFactory,          // (key, ctx) => new
    Func<TKey,TValue,TArg,TValue> updateValueFactory, 
    TArg factoryArgument);                           // the context
```

Almost similar situation is for the update. The update though, will require one more parameter: the previous value.

```c#
TValue AddOrUpdate<TArg>(TKey key,                   // the key
    Func<TKey,TArg,TValue> addValueFactory,          
    Func<TKey,TValue,TArg,TValue> updateValueFactory,// (key, prev, ctx) => new
    TArg factoryArgument);                           // the context
```

The synchronization of their execution is left up to the implementation of the dictionary. What you can be sure of, is that once this method returns, the returned value is the one that one of the methods set it to. Please notice: due to the use of the additional context parameter, we can use static lambdas again\! No context capturing.

# CancellationToken.Register

We’re slowly entering a much more demanding zone: the land of asynchronous programming. And one of the most important aspects of thriving in this land is to properly handle any cancellation that might occur. The main mechanism for handling aborting the ongoing operation in .NET is a CancellationToken. It has some basic properties that can be used to assert whether we should abort right now or if the cancellation is possible at all:

```c#
// some basic checks
CancellationToken ct;

ct.CanBeCanceled // can this token be ever cancelled? No? Don't bother
ct.IsCancellationRequested // Has it been already requested? Yes? Throw!
```

Now, with the basic options addressed, we need to cover how we can register for a cancellation happening. This means: the owner of the underlying source for cancellation requested an abort of the ongoing operation. To register for such an occasion, we use… Register method. Let’s try to register a TaskCompletionSource for such an event.

```c#
CancellationToken ct;
TaskCompletionSource tcs = new ();


ct.Register(() => tcs.TrySetCanceled());
```

We did it, didn’t we? Let’s try to add the mighty static modifier now to ensure that we don’t capture the scope.

```c#
CancellationToken ct;
TaskCompletionSource tcs = new ();

// Compilation error!
ct.Register(static () => tcs.TrySetCanceled());
```

It looks like we do capture the scope though and we can’t make it static\! Let’s recall the previous examples though. Usually, if the person who shaped the contract cares about the performance, they will provide an overload with the state. And this is the case here as well:

```c#
CancellationToken ct;
TaskCompletionSource tcs = new ();

// It can be static now!
ct.Register(
  static (state) => ((TaskCompletionSource)state).TrySetCanceled(), 
  tcs); // The state passed as the last parameter
```

Again, this requires some manual labor, as you need to cast down the state and pass it properly. Again, in an usual business application it might not be required, but if you’re building a database, you might use this technique to allocate a bit less. 

This is the case for [RavenDB](https://ravendb.net/?utm_source=blog.scooletz&utm_medium=link&utm_campaign=blogposts), a distributed database, which is built with .NET and uses the BCL (basic class library) `CancellationToken` to signal abort requests. PR [\#21205](https://github.com/ravendb/ravendb/pull/21205) introduces such improvements by using the state passing overload like we did above. Again, the mantra to follow is straightforward:

1. You notice a registration with `Register` that uses a non-static method  
2. You make the lambda static and wait for the compiler complaints  
3. If there is a single state, you use the state passing overload  
4. You profit.

Apply it in the hot paths, and your cancellation registrations just become so much cheaper. And please do remember: that if you’re building an infrastructure like RavenDB does, this stuff matters.

# Task.ContinueWith

A similar pattern to the `CancellationToken` can be used with continuations that can be attached to tasks manually. Sometimes you need to attach to a task without using the fancy async-await mechanism. In such a case you can use ContinueWith method:

```c#
Task connection;
TaskCompletionSource tcs;

connection.ContinueWith ( (t) => {
    if (t.IsFaulted)
        tcs.TrySetException(t.Exception);
    else if (t.IsCanceled)
        tcs.TrySetCanceled();
    else
        tcs.TrySetResult(null);
});
```

Again, we can follow the same approach by making the method static first. Then, notice the compiler error and fix it by the state passing overload and a cast down.

```c#
Task connection;
TaskCompletionSource tcs;

// The lambda is static now
connection.ContinueWith ( static (t, state) => {
    var c = (TaskCompletionSource) state; // cast down the continuation
    if (t.IsFaulted)
        c.TrySetException(t.Exception);
    else if (t.IsCanceled)
        c.TrySetCanceled();
    else
        c.TrySetResult(null);
}, tcs); // tcs passed as a state argument
```

To see this in action, please take a look at another RavenDB PR [\#21234](https://github.com/ravendb/ravendb/pull/21234). Again, we shaved off a few allocations here and there. This, if taken into consideration how optimized RavenDB codebase already is, can make a visible difference. So few allocations are left there to optimize them away.

# Summary

The closure capturing can be a real deal in hot paths of .NET execution. Fortunately, there are countermeasures in the form of static lambdas and the state passing overloads, that allow you to address these little breadcrumbs that are left on the table. And then, when there’s a lot less of them, [RavenDB](https://ravendb.net/?utm_source=blog.scooletz&utm_medium=link&utm_campaign=blogposts) can be even faster.
