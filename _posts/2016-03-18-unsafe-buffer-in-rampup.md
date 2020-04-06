---
layout: post
title: "Unsafe buffer in RampUp"
date: 2016-03-18 11:00
author: scooletz
permalink: /2016/03/18/unsafe-buffer-in-rampup/
nocomments: true
categories: ["RampUp"]
tags: ["RampUpNet", "unsafe"]
imported: true
---

As we're moving towards the core of [RampUp](https://github.com/Scooletz/RampUp), before visiting the most important parts we need to discuss one abstraction provided in the library named *IUnsafeBuffer*. The abstraction provides an interface a'la *Stream* wrapping a set of operations over a stream of an unmanaged memory. Currently, there's only one implementation using *VirtualAlloc*, but it's highly probable that in the future memory mapped files will be used as it's the easiest way of providing a cross-process visible memory. Now let's take a look at the following members of IUnsafeBuffer

[code language="csharp"]

public interface IUnsafeBuffer : IDisposable
{
    int Size { get; }
    unsafe byte* RawBytes { get; }

    AtomicLong GetAtomicLong(long index);
    AtomicInt GetAtomicInt(long index);
    void Write(int offset, ByteChunk chunk);
    void ZeroMemory(int start, int length);
}

```

As you can see the provided methods are a bit similar to the operations provided by the Stream. Currently pointer is leaked with RawBytes, but this is a subject to change. What's important, the unsafe buffer provides way to get [atomic wrappers](https://blog.scooletz.com/2016/03/10/atomic-in-rampup/) when needed. The atomics are designed to be used with any unmanaged memory provider, especially with IUnsafeBuffer. This API will be needed in creating the next data structure in RampUp
