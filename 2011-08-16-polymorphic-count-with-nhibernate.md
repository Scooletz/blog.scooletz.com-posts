---
layout: post
title: "Polymorphic count with NHibernate"
date: 2011-08-16 11:00
author: scooletz
permalink: /2011/08/16/polymorphic-count-with-nhibernate/
categories: ["NHibernate"]
tags: ["Cassandra", "Deiphobus", "NoSql"]
imported: true
---

If you're a user of NHibernate, I hope you enjoy using polymorphic queries. It's one of the most powerful features of this ORM allowing you to write queries spanning against several hierarchies. Finally, you can even query for the derivations of object to get all the objects from db (don't try this at home:P). The most important fact is, that having a nicely implemented *dialect* NH can do it in one DB call, separating specific queries by semicolon (in case of MS SQL)

[sourcecode language="csharp" light="true"]
// whoa!
var allObjects = session.QueryOver<object>().List();
```
Although the feature is powerful, you can find a small problem. How to count entities returned by the query without getting'em all to the memory. Setting a simple projection *Projections.RowCount()* will not work. Why? 'Cause it's gonna query each table with *COUNT* demanding at the same time that *IDataReader* should contain only one, unique result. It won't happen, and all you'll be left with it'll be a nice exception. So, is it possible to count entities in a polymorphic way? Yes! You can define in a extension method and use it every time you need a polymorphic count.

[sourcecode language="csharp" light="true"]
private static int GetPolymorphicCount(ISession s, Type countedType)
{
    var factory = s.GetSessionImplementation().Factory;
    var implementors = factory.GetImplementors(countedType.FullName);

    return implementors
        .Select(i => s.CreateCriteria(i)
                .SetProjection(Projections.RowCount())
                .FutureValue<int>())
        .ToArray() // to eagerly create all the future values for counting
        .Aggregate(0, (count, v) => count + v.Value); // sum up counts
}
```
Happy counting!
