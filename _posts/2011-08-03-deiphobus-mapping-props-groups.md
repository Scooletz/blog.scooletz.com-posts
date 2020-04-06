---
layout: post
title: "Deiphobus, mapping properties groups to column families"
date: 2011-08-03 11:00
author: scooletz
permalink: /2011/08/03/deiphobus-mapping-props-groups/
nocomments: true
categories: ["Deiphobus"]
tags: ["Cassandra", "NoSql"]
imported: true
---

As i it is stated in the [official documentation](http://wiki.apache.org/cassandra/DataModel "Data model"), the column family can be compared to a table in the relational database and as with table, the main target of creating one is to hold the same type of objects together to create a better model, allow querying, etc. Speaking about Cassandra it has one more advantage: the whole column family is stored on one server (the data are consistently hashed by key), so you may consider a column family as a collection of items frequently used together, for instance: the surname and the name, or the user name and the password.
In NHibernate you can embed those values within component to make it look like a whole, but they're still stored in the same table. Speaking about *Deiphobus*, it can be easily configured to match the needs of grouping properties (simple properties, and referencing other entities in many-to-one or one-to-one way) by implementing a convention interface:

[sourcecode language="csharp" light="true"]
public interface IPropertyFamilyConvention
{
    /// <summary>
    /// Gets the Cassandra family name.
    /// </summary>
    /// <param name="mappedType">The mapped type.</param>
    /// <param name="propertyInfo">The info of a mapped property.</param>
    /// <returns>The family name.</returns>
    FamilyName GetFamilyName(Type mappedType, PropertyInfo propertyInfo);
}
```

It's worth to mention, that by default all the properties are mapped to one column family called *'Entity'*. Ok, you know how it influences the storage from the Cassandra point of view, but what about Deiphobus? As it was implemented, every time you access previously not loaded property of an entity mapped with Deiphobus, the whole column family, containing the specified property is loaded from the database (actually a bit more data is retrieved, but for the sake of simplicity it can be omitted in here). It means, that once you started using a property which is strongly connected with others, all the needed properties will be loaded in one db hit. Simple and powerful, isn't it?
