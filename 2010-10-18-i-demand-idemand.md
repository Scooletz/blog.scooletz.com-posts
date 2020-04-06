---
layout: post
title: "I demand IDemand"
date: 2010-10-18 20:00
author: scooletz
permalink: /2010/10/18/i-demand-idemand/
nocomments: true
categories: ["Architecture", "Design patterns", "NHibernate", "Themis"]
tags: ["architecture", "authorizations", "design patterns"]
imported: true
---

The main interface of [Themis](http://themis.codeplex.com/ "Themis") is IDemand.

```csharp
public interface IDemand
{
}
```

An example implementations:

```csharp
public class EntityPermission : IDemand
{
    public EntityPermission(TEntity entity)
    {
        Entity = entity;
    }

    public TEntity Entity { get; private set; }
}

public class View : EntityPermission
{
    public View(TEntity entity)
        : base(entity)
    {
    }
}
```

A simple interface, with no members at all. The class implementing it defines the demand name as well as the result which should be provided by the system evaluating the demand. Implementations of IDemand should be contextful, passing all the needed context for the demand evaluation in their properties. The TResult can be any type, but in majority of authorization cases it'll be nothing more then bool. We want a yes/no answer for 'Can I?' question, don't we? :)

Having the defined roles and demands we can move to the UI and write a simple authorization check. Being given an instance of IDemandService (by constructor injection), I can write in my ASP MVC controller a simple check of a demand/authorization. Below you can find the signature of IDemandService, a simple extension method written to ease using the service in a specific domain (a wrapping service is also a good solution) and the usage example itself.

```csharp
public interface IDemandService
{
    IEnumerable<TResult> Evaluate<TDemand, TResult>(TDemand permission, params object[] roles)
        where TDemand : IDemand<TResult>;
}
```

Useful extension methods written in the app using Themis. The names, I hope, are self-descriptive.

```csharp
    public static class AuthorizationServiceExtensionMethods
    {
        public static bool Can<TDemand>(this IDemandService demandService, TDemand permission,
            params object[] roles)
            where TDemand : IDemand<bool>
        {
            return demandService
                .Evaluate<TDemand, bool>(permission, roles)
                .Any(b => b);
        }

        public static bool CanEdit<TEntity>(this IDemandService demandService, TEntity entity,
            params object[] roles)
        {
            return demandService
                .Evaluate<Edit<TEntity>, bool>(new Edit<TEntity>(entity), roles)
                .Any(b => b);
        }

        public static bool CanView<TEntity>(this IDemandService demandService, TEntity entity,
            params object[] roles)
        {
            return demandService
                .Evaluate<View<TEntity>, bool>(new View<TEntity>(entity), roles)
                .Any(b => b);
        }
    }
```

And finally, the app code example in which you can see that role management was delegated to an enigmatic IRoleService. The 'Should I use NH session to manage roles?' question is out of scope of this entry.
```csharp
public ActionResult View(Guid id)
{
    var car = _session.Get<Car>(id);
    var roles = _roleService.GetCurrentUserRoles();
    if (!_demandService.CanView(car, roles))
        throw new InvalidOperationException("You cannot view this car!");

    return View(car);
}
```

Isn't is simple? What about the expressions and the configuration? I'll cover it soon.
