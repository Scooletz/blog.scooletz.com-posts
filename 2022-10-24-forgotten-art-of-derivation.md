---
layout: post
title: "The Forgotten Art of C# Inheritance"
date: 2022-10-24 07:00
author: scooletz
permalink: /2022/10/24/forgotten-art-of-c-sharp-inheritance
image: /img/2021/06/csharp.png
categories: ["csharp", "dotnet", "inheritance"]
tags: ["csharp", "dotnet", "inheritance"]
whitebackgroundimage: true
twitter: true
---

Every inheritance post should start with a question about a nature of things, like what is what, whether a cat is an animal, etc. Then it should be followed by some other post praising the mighty composition over inheritance as the one design principle. Fortunately, this post is (pun intended) neither of them. We'll focus solely on the data oriented aspect of inheritance. Let's go.

### Inheritance

In C\#, a class can inherit only from one class.

```csharp
class Animal {}

class Cat : Animal { }
```

There's no C++ like multi-inheritance. Of course, if one wanted to cheat a bit `default interface methods` could be potentially used to "share" some implementation. Still, a class can inherit only from one class.

### Composition over inheritance

The mentioned earlier _composition over inheritance_ is often sold as a kind of panacea.
Why to answer whether X is A or B, where X can be composed from needed attributes. What if we'd like to design something that combines cat's properties and is capable of shooting a laser gun?

```csharp
class X {
  private readonly Cat _cat;
  private readonly LaserGunShooter _shooter;
}
```

Then it would be up to `X` implementor to surface needed methods, so that their users can refer to cat's aspects or laser gun shooter.

### Composition vs data orientation

Even though composition can be pleasing, as one does not need to make the final decision what a component is, it can introduce some potential memory/data/performance penalties.

The first one is an additional reference jump. After all if `Cat` and `LaserGunShooter` are reference types, they are allocated on the heap and accessing a `_cat` or the other field requires following a pointer/reference to another memory region. Sometimes, it might introduce an overhead.

If `Cat` and `LaserGunShooter` are reference types, then their instances will have an object header. An object header is an additional payload added to each object that is of a reference type. It contains information about the object type and a few more items. It can be thought of as a few additional hidden fields that are added to every single class that is declared out there.

Now, you could say that `Cat` and `LaserGunShooter` could be defined as `struct`. The struct does not have any of the two disadvantages mentioned above. It requires no additional jump, because it's stored where it is. Also, it has no object header, it has this value semantics.

This approach of turning components into structures is quite common when you work with codebase that requires robust performance. They can be embedded in others without any overhead. This is not always possible though.

### Togetherness with other constructs

One of the common tasks that you can tackle, when working with some low level types, is interacting with them and passing an additional context. Let's consider an example of `TaskCompletionSource<T>`. `TaskCompletionSource<T>` is a class providing a capability of managing a lifecycle of a `Task<T>`. If you want to have a possibility to create a `Task<T>` that later can be cancelled or have it state set, this is the way to go.

```csharp
var tcs = new TaskCompletionSource<int> ();
return tcs.Task;

// later, when the output is known
tcs.SetResult(5);

// or if it should throw an exception
tcs.SetCancelled(new Exception());
```

Now, if in a given scenario `TaskCompletionSource<int>` is used, it will require capturing it somewhere. Usually, it will require capturing some additional information as well. This can be done in many ways. One can use a `Dictionary<TKey, TValue>`, tuples or other approaches. Are there other approaches?

### Derivation in data orientation for Durable Functions

One of the examples from my work for using derivation in data orientation are [Durable Functions' events made twice as fast](https://blog.scooletz.com/2021/09/27/durable-functions-events-made-twice-as-fast). In this PR, I introduced a change that instead of encapsulating `TaskCompletionSource<T>` or using an assisting data structure to capture additional context, it used the derivation for this.

```csharp
private class EventTaskCompletionSource<T> : 
  TaskCompletionSource<T>,   // derive from tcs, no additional jumps needed, 1 object header instead of 2
  IEventTaskCompletionSource // slam an additional interface on top of it
{
  public Type EventType => typeof(T);

  // one additional field to construct a stack from TCSs objects 🤯
  public IEventTaskCompletionSource Next { get; set; }

  // this.TrySetResult as this is the task completion source!
  void IEventTaskCompletionSource.TrySetResult(object result) 
    => this.TrySetResult((T)result);
}

// A non-generic tcs interface.
private interface IEventTaskCompletionSource
{
  // The type of the event stored in the completion source.
  Type EventType { get; }

  // The next task completion source in the stack.
  IEventTaskCompletionSource Next { get; set; }

  // Tries to set the result on tcs.
  void TrySetResult(object result);
}
```

Effectively, there's one object here - an extended `TaskCompletionSource<T>` that can be added with any additional fields for making it work. It provides no additional memory overhead, as the completion source is an object on it own and embraces all the required properties and functionalities.

### Summary

The derivation is often treated as the worst evil nowadays. When used properly, it help in building efficient solutions in regards to the data orientation. This can be done especially, when working with types that are already reference types and they almost work and require just a bit of additional data.