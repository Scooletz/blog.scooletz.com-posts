---
layout: post
title: "Themis, a bit dipper dive"
date: 2011-02-27 20:00
author: scooletz
permalink: /2011/02/27/themis-a-bit-dipper-dive/
nocomments: true
categories: ["Themis"]
tags: ["authorization"]
imported: true
---

I just committed some new code for [Themis](http://themis.codeplex.com/) examples, which for now on, will be also an integration tests. The whole case is about values which are allowed for some roles.

Let's take a case where there are two types of users: Admins and standard users, for the sake of briefness called Users. Let Admins are allowed to choice an offer type between Internal and External, and a standard user's choice is narrowed to one value: Internal. How can a domain be modeled to handle it easily? First of all roles are needed:
```csharp
public class AdminRoleDefinition : RoleDefinitionBase<Admin>
{
    public AdminRoleDefinition( )
    {
        ValueIsAllowed(OfferType.Internal);
        ValueIsAllowed(OfferType.External);
    }
}

public class UserRoleDefinition : RoleDefinitionBase<User>
{
    public UserRoleDefinition( )
    {
       ValueIsAllowed (OfferType.Internal);
    }
}
```
What's the method *ValueIsAllowed*? It's a simple, internal extension method, setting the possible value, using an *markup* demand.

```csharp
public static class ServiceExtensions
{
    public static TValue[] GetAllowedValues<TValue>(this IDemandService @this, params object[] roles)
    {
        return @this.Evaluate<AllowedValueDemand<TValue>, TValue>(
                AllowedValueDemand<TValue>.Instance, roles);
    }
}

internal sealed class AllowedValueDemand<TValue> : IDemand<TValue>
{
    public static readonly AllowedValueDemand<TValue> Instance = new AllowedValueDemand<TValue>();
}
```
Having all this configured, now one can query a demand service for allowed values for a specific drop-down! It could be also done on [the filtering basis with NHibernate](http://themis.codeplex.com/wikipage?title=NHibernate%20integration), but it was an example how simply extensible Themis is.
