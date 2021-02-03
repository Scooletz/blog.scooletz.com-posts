---
layout: post
title: "My mental model of Span, Memory and ReadOnlySequence in .NET"
date: 2021-02-03 11:00
author: scooletz
permalink: /2021/02/03/spans-memory-mental-model
image: /img/2021/02/memories-of-t.png
categories: ["dotnet", "csharp", "memory"]
tags: ["dotnet", "csharp", "memory"]
whitebackgroundimage: true
---

Saying that I disagree with the documentation of the new memory abstractions in .NET would be an overstatement. After working a bit with `Span`, `Memory`, `ReadOnlySequence` and `IBufferWriter` I model it in my head in a bit different way. Each of these terms has a one line definition that one could reason from. I hope that newcomers to this memory friendly world will think of it a good primer. At the same time, I hope it delivers a refreshing view for more experienced engineers. It does not aim to deliver the whole description. It aims to be a useful model as all models are wrong but some of them are useful.

### Span - a fast synchronous accessor

I treat `Span<T>` as a fast synchronous accessor of a continuous chunk of memory. As simple as that. When thinking about it, I tend to abstract away the source of the memory and focus on its usage. A `Span<T>` can be used in synchronous code (meaning: no `async await`) . Its usage usually indicates that a piece of code should be fast and performing well.

It's worth to mention that using `foreach` with it is also fast. It is based on a span's non-allocating ref-returning implementation of `GetEnumerator`. It's almost like it was designed to be really fast ;-)

Having in mind this part of the model, that it's a fast synchronous memory accessor, it's good to ask a good old fashioned question: why? Why does it have to be a synchronous accessor? What's the limitation behind it? To answer this let's list some of the sources for a continuous chunk of memory behind the span:

1. a slice of some array `T[]`
1. some `Memory<T>`
1. unmanaged pointer `void*`
1. `stackalloc`

The first is as simple as possible, just a slice of an array allocated on the `heap`. The second is based on `Memory<T>`, but for now, let's take a leap of faith with this one. Using pointers like `void*` provides a way to interact with the unmanaged world, but still does not infer that it's synchronous. What about the last one, `stackalloc` then?

`stackalloc` provides you the way to allocate a slice of memory on the stack of a thread that is being executed. Let's assume that `Span<T>` could be used in an asynchronous scenario. What would happen when a continuation of an async call is executed by another thread? It would result in one thread having an access to the stack of another thread. This is not how it's supposed to be. Keeping `Span<T>` a synchronous prevents this kind of behavior.

To sum it up, `Span<T>` is a fast synchronous accessor of a continuous chunk of memory. It's not the memory, it's just a really performance friendly view of it.

### Memory - an actual memory chunk

The `Memory<T>` is an actual continuous memory chunk. It can be passed in asynchronous flows. It provides a way to get the efficient synchronous accessor of it - `Span<T>`. It can be based on different sources as well. Let' consider the following examples

1. a slice of some array `T[]`
1. `MemoryMarshal.Create*` methods like `MemoryMarshal.CreateFromPinnedArray`

The first one is again the basic scenario. Take an array of `T[]` and use a slice of it as `Memory<T>`. The second is a bit more complex as it allows to build a memory in a special way. In the provided sample it uses an already pinned array (`pinning` in .NET prohibits Garbage Collector from moving an object and can be useful when passing memory to the unmanaged world). Where can it be found?

Let's take a look at `AspNetCore`. It uses it provides [a custom memory pool](https://github.com/dotnet/aspnetcore/tree/main/src/Shared/Buffers.MemoryPool) that provides a pool for 4kb blocks of memory. All blocks are create from large slabs of memory that is already pinned. Therefore there's no need for pinning it again whenever a memory is pinned/unpinned.

To sum it up, `Memory<T>` embeds an actual memory chunk, that can be passed wherever you want and accessed using its fast synchronous accessor `Span<T>`. When needed it can be pinned to obtain the pointer, but most likely, you will `.Span` it.

### ReadonlyX - is a X but readonly

Both, `Span<T>` and `Memory<T>` have their readonly counterparts: `ReadOnlySpan<T>` and `ReadOnlyMemory<T>`. As the span is an synchronous accessor for the memory, the readonly span is an accessor for the readonly memory.

### ReadOnlySequence - a sequence of ReadOnlyMemory elements

Sometimes a memory doesn't come in one piece and is shattered. Still, it'd be useful to have a construct that can represents a chain, a list, a sequence of multiple pieces that represent one thing. This is the reason why
`ReadOnlySequence<T>` was introduced. It's a list of `ReadOnlyMemory<T>`. It's optimized for cases where the sequence contains one element by providing properties like:

1. `IsSingleSegment` - the fast check whether it contains just one memory item
1. `FirstSpan` - the fast access to the `ReadOnlySpan<T>` accessor to the first memory

The reason for this special case is following. Even when designed as a sequence, it's much easier to deal with a single element and optimize for "one span/memory" scenario.

To sum it up, `ReadOnlySequence<T>` is a linked list of `ReadOnlyMemory<T>` elements. It provides a special case properties for its convenient usage where it contains a single element.

### Summary

So here were are. At the end of this model. Let's sum it up!

1. `Span<T>` - a fast synchronous accessor of a continuous chunk of memory. It's not the memory, it's just a really performance friendly view of it.
1. `Memory<T>` - an actual memory chunk, that can be passed wherever needed and accessed using its fast synchronous accessor `Span<T>`.
1. `ReadOnlySpan<T>` - a span but readonly
1. `ReadOnlyMemory<T>` - a memory but readonly
1. `ReadOnlySequence<T>` - a linked list of `ReadOnlyMemory<T>` elements. It provides a special case properties for its convenient usage where it contains a single element.
