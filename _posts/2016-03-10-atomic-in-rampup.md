---
layout: post
title: "Atomic* in RampUp"
date: 2016-03-10 11:00
author: scooletz
permalink: /2016/03/10/atomic-in-rampup/
nocomments: true
categories: ["Optimization", "RampUp"]
tags: ["RampUpNet", "threading"]
imported: true
---

In the two recent posts we've build strong/relaxed foundations to take a very first look at some parts of [RampUp](https://github.com/Scooletz/RampUp) library.

### Structs vs classes

The differences seem to be obvious. Structs are allocated on the stack (they are on heap when allocated in arrays), are passed by value unless passed by *ref* or *out*, should be small as they are copied by value as well. Instances of classes are allocated on the heap, are passed by reference. If we considered a proper wrapper for all the operations related to the value of a *long*, including operations related to the class Volatile, we'd need to use a class, as a struct would be copied, hence referring to the same address would be impossible, right? Not exactly

### Hello pointer

Let's take a look at the [AtomicLong](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Atomics/AtomicLong.cs) implementation now. It's a struct. Why can it be a struct? Oh, wait a second. It doesn't have a *long* field representing the value. It has a *long** field, representing the address of the value! Now, when the AtomicLong is copied, it has the address copied, not the value, hence, it preserves the value and all the operations that were issued against it. How one can obtain a pointer to long which will be valid forever? It can be done in many ways, but one of them is to allocate an offheap, unmanaged memory using for instance [Marshal.AllocHGlobal](https://msdn.microsoft.com/en-us/library/s69bkh17%28v=vs.110%29.aspx). RampUp does it in a different way (still, using unmanaged memory) but the result is the same. You have a nice wrapper around all the operations for a single long gathered in one place.

### Ref vs pointer

The most interesting parts of this <del>class</del> struct are methods using *ref*. As you can see below to use [Volatile.Read](https://msdn.microsoft.com/en-us/library/gg712759%28v=vs.110%29.aspx) the pointer is first dereferenced and then, the reference to the value is taken. How does it work?

[code language="csharp"]
[Pure]
public int VolatileRead()
{
    return Volatile.Read(ref *_ptr);
}
```

To answer this question, you can compile RumpUp and use an IL viewer (DotPeek, ILSpy, Ildasm, whatever you want). You'll see an interesting thing. The pointer is passed to the method without any modifications, in other words, dereferencing and then taking a reference to the obtained value negate each other resulting in the pointer being passed to the *Volatile.Read*.

### Summing up

This simple wrapper around all the memory & memory barriers related operations for *long* is one of the pillars of RampUp. Now we can move forward with more advanced operations.
