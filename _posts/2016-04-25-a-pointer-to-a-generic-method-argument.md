---
layout: post
title: "A pointer to a generic method argument"
date: 2016-04-25 09:00
author: scooletz
permalink: /2016/04/25/a-pointer-to-a-generic-method-argument/
nocomments: true
categories: ["C#", "RampUp"]
tags: ["generics", "RampUpNet"]
imported: true
---

Let's consider a following method signature of an interface taken from a [RampUp interface](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Actors/Impl/IMessageWriter.cs).

```csharp

bool Write<TMessage>(ref Envelope envelope,
    ref TMessage message, IRingBuffer bufferToWrite)
    where TMessage : struct;

```

It's a fairly simple signature, enabling to pass a struct of any size using just a reference to it, without copying it. Now let's consider the need of obtaining a pointer to this message. Taking a pointer could be needed for various reasons. One could be getting fields by offset, another could be using *memcpy* for copying the value to any given address. Is it possible to get this pointer in C# code?

### No pointers for generic parameters

Unfortunately, you can't do it in C#. If you try to obtain a pointer to a generic parameter, you'll be informed about the compiler error. If you can't do it in C#, is there any other .NET language one could use to get it? Yes, there is. It's the foundation of .NET programs, the MSIL itself and if it's MSIL, it means emitting code dynamically.

### Ref looks like a pointer

What is a reference to a struct? It looks like a pointer to me. What if we could load it and just assume that it is a pointer? Would CLR accept this program? It occurs that it would. I won't cover the whole implementation which can be found in [here](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Actors/Impl/MessageWriterBuilder.cs#L88), but want to accent some points.

* CLR uses the argument with index 0 to passing *this*. If you want to load a field you need to use the following sequence of operations:

    *   Ldloc_0; // load this on the stack

    *   Ldfld, "Field1" // pops this loading the value named "Field1" on the stack
* For Write method, getting a pointer to a message is nothing more than calling an op code: ***Ldarg_2**.* As the struct is passed by reference, it can be treated as a pointer by CLR and it will.

I encourage you to download the [RampUp](https://github.com/Scooletz/RampUp) codebase and play a little bit with an emitted implementation of the *IMessageWriter*. Maybe you'll never need to take the pointer to a generic method parameter (I did), but it's a good starter to learn a little about emitting code.
