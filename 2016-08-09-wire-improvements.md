---
layout: post
title: "Wire improvements"
date: 2016-08-09 09:20
author: scooletz
permalink: /2016/08/09/wire-improvements/
nocomments: true
categories: ["C#", "Open Source"]
tags: ["Akka", "OSS", "Wire"]
imported: true
---

**TL;DR**

This post sums up my recent involvement in the [Wire serializer](https://github.com/akkadotnet/Wire) being built by the Akka.NET team. It was an interesting OSS experience. I'm glad that I could improve the performance of this serializer .

### Box me not

The initial design of constructing a serializer of a complex object was based on emitting a pair of delegates (serialize, deserialize). Let's consider deserialization:

1. Create object
1. For each field:

    1.  Use the following method of a **ValueSerializer:* *

```csharp
public abstract object ReadValue(
   Stream stream, DeserializerSession session)
```

    2.  Cast to the field type or ubox if the field is a value type

    3.  Load the deserialized object and set the field.

The problem with this approach was manifesting for classes with many primitive properties/fields the cost of boxing/unboxing might be quite high. On the other hand, one couldn't be calling a method returning a common type without boxing for value types. How would you approach this? My answer was to move emitting point 1 & 2 to the value serializer and provide a generic implementation for this emitting in the basic class. That allowed me to customize the emitting for primitive value types like *int*, *long,* *Guid* still preserving a possibly bit less performing generic approach.

After the change and delegating the emitting to the *ValueSerializer* class, it was given two new virtual methods with their implementation using boxing etc, but leaving much more space for special cases

```csharp
public virtual int EmitReadValue(Compiler<ObjectReader> c,
   int stream, int session, FieldInfo field)
public virtual void EmitWriteValue(Compiler<ObjectWriter> c,
   int stream, int fieldValue, int session) ```

### Call me once

Wire uses a BCL's *Stream* abstraction to work with a stream of bytes. When using a stream, a conversion to primitive value types sometimes require a helper byte array to get all the data in one Read call. For instance, a *long* value is stored as 8 bytes, hence it requires an 8 byte array. To remove allocations, during serialization and deserialization, a byte chunk can be obtained by calling *GetBuffer* method on the session object. What if your object contains 4 longs. Should every serializer call this method or maybe it should be called once and the result stored in a variable?

I took the second approach and by removing this additional calls to *GetBuffer* was able to squeeze again a bit more from Wire.

### Summing up

Working on these features was a very pleasant experience. It took me just a few evenings and I was able to make a positive impact. And this is really nice.
