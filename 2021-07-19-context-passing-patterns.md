---
layout: post
title: "Context passing patterns"
date: 2021-07-19 11:00
author: scooletz
permalink: /2021/07/12/context-passing-patterns
image: /img/2021/07/context.png
categories: ["dotnet", "csharp", "programming", "dev"]
tags: ["dotnet", "csharp", "programming", "dev"]
whitebackgroundimage: true
---

_It just gets injected!_

This is a standard answer of a dependency injection fan to any question related to dependencies. But there are many kind of them, like repositories, services, factories. The one that is often omitted is the context. And there's a good reason to why it is not discussed! It's frequently filled with raw data and all the dependencies that didn't fit other criteria. Still, even as a bag of things, it's sometimes vital for writing a succinct snippet that implements a specific functionality. Let's discuss all the context passing patterns.

### Method Parameter Binding

The first pattern for passing the context could be shared in the following manner:

```csharp
void Order(OrderCommand orderProduct, IContext context);
```

It is often seen in the highest level components embedded by frameworks supporting `binding` mechanism. Think about `ASP.NET` with their controllers or `Azure Functions` with its... functions. These frameworks provide multiple levels on which you can build your solution, for example supporting the mechanism for resolving the specific value of a parameter. This is possible as there's no `interface` that the method signatures must adhere to. It can be leveraged to create code that is:

- much more cohesive
- easy to migrate elsewhere

The first is related to the fact, that parameters used locally does not blow up the component (big no for controllers having tens of ctor parameters). There are actually no dependencies injected, only values bound. The code is also easy to migrate elsewhere. After all, if the function requires only parameters, it can be made `static` and moved outside of the component.

### Interface Method Parameter

The second approach is related to either a third-party libraries/frameworks or the infrastructure code you write on your own. If you use a framework, it's quite common that it provides a set of contracts that need to be implemented.

```csharp
public class Handler : IHandle<OrderCommand>
{
    void Order(OrderCommand orderProduct, IContext context)
    { 
        // TODO: implement it
    }
}
```

This approach is quite common in C&#35; based libraries which like to be type oriented. It's quite uncommon to see it being function based, which would allowed a bit more relaxed resolution. The interface method parameter also has some disadvantages as it requires to match the interface, meaning, that you should follow the number and the types provided by the framework. It's ok if you're design approach is aligned with the author of the infrastructure code, but if it isn't you may find it hard to match the required interface.

### Dependency Injection

Finally! The good old fashioned dependency injection is here!

```csharp
public class Service
{
    readonly IContext context;

    public Service(IContext context)
    {
        this.context = context;
    }

    void Order(OrderCommand orderProduct, IContext context)
    { 
        // TODO: implement it
    }
}
```

If the assumption is that the service lives not longer than the context is useful (consider a singleton with the `.Now` provided in the context), this is fine. Depending on the injected context usage, the component might be not cohesive though. What if only one out of 10 methods uses the context? What about the rest? Should they be extracted to a separate component?

### Static Context Provider

If you ever worked with old fashioned `ASP.NET` you probably used `HttpContext.Current` a lot of times. I'm guilty of it! This is the pattern, that allows you to reach out for context whenever needed, by accessing the context using a `static` property or a `static` getter method.

```csharp
public static class ContextProvider
{
    public static IContext Current { get; }
}
```

This is an easy way to introduce the context to an existing application. At the same time, it might make your application use some magically delivered objects, that cannot be reasoned about when looking at methods' and constructor's parameters. This is why some projects prohibit using the most abused one: `DateTime.Now`.

The static context provider should also be aware of `async await` and all the underling infrastructure. `[ThreadStatic]` in modern .NET might not do its job.

### Static Context Extension

The last patter is `Static Context Provider` dressed as something else. Instead of having the `.Current` property, one could use an extension method that accepts either a specific type, or any `object` at all.

```csharp
public static class ContextProvider
{
    static readonly ConditionalWeakTable<object, IContext> _ctw;

    public static IContext GetCurrent(this object @this)
    {
        if (_ctw.TryGetValue(@this, out var context))
            return context;
        
        // TODO: implement
    }
}
```

This approach, could benefit from attaching specific contexts to specific objects, using for example [ConditionalWeakTable](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.conditionalweaktable-2?view=net-5.0), where the object itself is the key for the table. Beside being more specific, it still requires knowing that the context should be used and again, it won't be visible just by looking at the parameters passed to the component.

### Summary

This is a list of patterns that you may find when working with various codebases and libraries/frameworks. This is not a definitive list and I'm far from attaching bad/good label to any of these. Depending on the stage of the project, the maturity of the team some of them might be more useful than the rest. As always remember, you can evolve your codebase into any direction. Just name clearly the pattern behind it and know your direction.
