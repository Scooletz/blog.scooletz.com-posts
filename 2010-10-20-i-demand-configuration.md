---
layout: post
title: "I demand configuration"
date: 2010-10-20 20:00
author: scooletz
permalink: /2010/10/20/i-demand-configuration/
nocomments: true
categories: ["Architecture", "Design patterns", "Themis"]
tags: ["architecture", "authorizations", "design patterns"]
imported: true
---

In the previous posts I described the intentions behind [Themis](http://themis.codeplex.com/ "http://themis.codeplex.com/"). It's time do deepen into demand and the whole configuration. Imagine a domain of car drivers. It includes car drivers as well as driven cars. The simplest demand which one can come up with is a simple check, whether one can drive a specific car.
```csharp
public class CanDrive : IDemand<bool>
{
    public CanDrive(Car car)
    {
        Car = car;
    }

    public Car Car { get; private set; }
}
```
Speaking about a car and car driver:
```csharp
public class DriverRole
{
    public DriverRole(int skillLevel)
    {
        SkillLevel = skillLevel;
    }

    public int SkillLevel { get; private set; }
}

public class Car
{
    public Car(int requiredSkillLevel)
    {
        RequiredSkillLevel = requiredSkillLevel;
    }

    public int RequiredSkillLevel { get; private set; }
}
```

The last but one thing, which should be done is to create a defintion of the DriverRole. By creating a definition I mean, creating a set of rules applied to evaluated demands in scope of the role context:
```csharp
public class DriverRoleDefinition : RoleDefinition<DriverRole>
{
    public DriverRoleDefinition( )
    {
        Add<CanDrive, bool>((d, r) => d.Car.RequiredSkillLevel <= r.SkillLevel);
    }
}
```
Having all of this, during application startup one can easily configure the demand service with fluent configuration:
```csharp
var demandService = Fluently.Configure()
    .AddRoleDefinition(new DriverRoleDefinition())
    .BuildDemandService();
```
and use the created service to evaluate demand *canDrive* created on a retrieved car basis, passing the driver role as the argument

```csharp
return _demandService.Evaluate<CanDrive, bool>(canDrive,  driverRole).Any(b=>b);
```
In the solution, I created a more meaningful example (Themis.Example project), showing how with simple extension methods get rid of plenty of generic parameters.

In the next posts I'll write about Themis extension points and describe a way in which I want to integrate it with NHibernate.
