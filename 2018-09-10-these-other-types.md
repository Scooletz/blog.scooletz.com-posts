---
layout: post
title: "These other types and your API"
date: 2018-09-10 09:54
author: scooletz
permalink: /2018/09/10/these-other-types/
image: /img/2018/09/api.jpg
categories: ["Design"]
tags: ["API", "design"]
imported: true
---

So you've created your Open Source project. It might be written in Java, it might be .NET, it might be something else. You might have even taken an additional step and created a test that validates public API to please SemVer gods. Now, let's take a closer look at this API. What types does it use and why?

### List me not

Let's consider a simple example. A C# method of an interface that can be implemented by a developer who uses our library.

```csharp

public interface ICanDoIt
{
public void DoIt (List<T> data);
}
```

Who wouldn't like to implement *ICanDoIt*, huh? This is the best interface ever! So every single user of your library does it. In their implementations, they add to the list, they remove from the list, they augment it in every single way you can imagine.

At this moment everything works just fine. But a few months later, you want to slightly alter the the behavior of this interface. You'd like to track all the operations on the *data* parameter to allow, for instance, rolling back changes, when needed. How can this be implemented?

### I can't do it

It can't be. *List<string>* has no capability like this. In .NET, where methods are not virtual by default, there is no way to derive from the *List<T>* to add the behavior specified above. What went wrong then?

The author of the library attached its public API not to the contract/behavior of the list but to its implementation. Now, every single user can use ANY part of the implementation, not only the part that we meant. This means that it can't be easily swapped. You could think of different ways of moving it like:

1. Replace with the widest interface like *IList<T>*, which breaks backward compatibility, changing the signature of the method.
1. Introduce a new interface, that will break it even more.
1. Try to forward the type. With a Basic Class Library type it won't go that easy.

As you can see, initial attachment to the implementation introduced a lot of harassment when it required a change.

### No strings attached

You can't develop your library in vacuum. After all, you want to live in the ecosystem you build your library for. But using all the types from a standard library might not be the best way to build a SemVer friendly product. Next time, when you thing about using *List<>* or a *Dictionary<,>* think again. Maybe, it's not the best choice you can make. Maybe, using a smaller interface that allows augmenting behavior later on, is the way to go.
