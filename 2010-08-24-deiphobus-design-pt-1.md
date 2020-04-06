---
layout: post
title: "Deiphobus design, pt. 1"
date: 2010-08-24 19:00
author: scooletz
permalink: /2010/08/24/deiphobus-design-pt-1/
categories: ["Databases", "Deiphobus"]
tags: ["databases"]
imported: true
---

It's the right time to write about Deiphobus design. I'll start with an example of usage, next I'll move to configuration, serialization and model created from the config. In the next topic the event design of session implementation, the identity map and its usage will be explained as well as the lifetime of a session and query API.

The configuration provided with Deiphobus is a fluent, "just code" configuration. Each entity is described with a generic *EntityClassMap*.

```csharp
/// <summary>
/// The mapping of the class, marking it as entity, stored under separate key.
/// </summary>
/// <typeparam name="T">The type to be mapped.
public abstract class EntityClassMap<T>
 where T : class
{
 protected void Id(Expression<Func<T, Guid>> id)
 protected IndexPart IndexBy(Expression> memberExpression)
 protected void SetSerializer<TSerializer>( )
 where TSerializer : ISerializer
}
```

The class interface was designed to be similar to [Fluent NHibernate](http://fluentnhibernate.org/):

* class specifying this type, should be a mapped entity class
* *Id* method marks the specific property as id. It's worth to notice, that only Guid identifiers are available
* the second method is *IndexBy* used for marking a property to be indexed with an inverted index. Only the properties marked with this method can be queried in Deiphobus queries. Running query on a not indexed property will throw an exception
* the very last method, allows to set a custom serializer type for the mapped entity type

All the mappings are consumed by mapping container registering all entity class maps in itself. The maps are translated into *EntityClassModel* object, describing the specific entity properties. This process takes place when the session factory is created. On the basis of each model class, the object implementing the interface *IEntityPersister* is created. The implementation of the persister provides methods like: *GetPropertyValue* or *GetIndexedPropertyValues* with IL code emitted, to overcome the reflection overhead. This class will be described later, the *EntityClassModel*'s method signatures can be seen below:

```csharp
/// <summary>
/// The class representing a model of mapped entity.
/// </summary>
public class EntityClassModel
{
 public EntityClassModel(Type classType, PropertyInfo id, object idUnsavedValue, IEnumerable<IndexedProperty> indexedProperties)
 {
  ClassType = classType;
  Id = id;
  IdUnsavedValue = idUnsavedValue;
  IndexedProperties = indexedProperties.ToList().AsReadOnly();
 }
 public Type ClassType { get; private set; }
 public PropertyInfo Id { get; private set; }
 public object IdUnsavedValue { get; private set; }
 public IEnumerable<IndexedProperty> IndexedProperties { get; private set; }
 public Type SerializerType { get; set; }
}
```

The very last part of this entry, is for serialization in Deiphobus. Because of the usage of Cassandra, each entity is stored under one key, in one column family, in one column. The entity is serialized in the moment of storing. The serialized entity is stored in Cassandra as well as its inverted indexes based on values retrieved just before saving the entity in the database. In the current moment, two levels of serializers can be setup:

* the default, used by all classes not having their own
* entity class specific

The rest of types is always serialized using the default serializer. This behavior may be subject to change.
