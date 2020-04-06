---
layout: post
title: "Pearls: the protobuf's discriminated union"
date: 2018-01-25 09:55
author: scooletz
permalink: /2018/01/25/pearls-the-protobufs-discriminated-union/
nocomments: true
image: /img/2018/01/stencil-default3.jpg
categories: ["Design", "Optimization"]
tags: ["design", "pearls", "Protobuf-net", "serialization"]
imported: true
---

Google Protocol Buffers is a proven protocol for serializing data efficiently. It has a wide adoption, enabling serialization for almost every platform, making the data easy to exchange between platforms. To store its schema, you can use .proto files, that enable describing messages in a platform agnostic format. You can see an example below:

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

### One of

Sometimes you want to define a message that will have only one of its fields initialized. This can be useful when sending a message providing a wrap around multiple types or in any other case that requires it. Take a look at the example, providing a wrap around three messages of the following types: MessageA, MessageB and MessageC.

<span class="pln">message </span><span class="typ">Wrapper</span> <span class="pun">{</span><span class="pln">
  oneof OnlyOneOf </span><span class="pun">{</span><span class="pln">
    </span><span class="kwd">MessageA a = 1;
</span><span class="pln">    MessageB b = 2;
    MessageC c = 3;  </span><span class="pln">
  </span><span class="pun">}</span>
<span class="pun">}</span>

Protocol buffers use a tag value to store any field. First, the tag (1, 2, 3 above) and the type of the field is stored, next, the value is written. In the following case, where only one field is assigned, it would write its tag, type and value. Do we need to create a class that will contain all the fields? Do we need to waste this space for storing fields?

### Protobuf-net Discriminated Union

To fix this wasteful generation, [protobuf-net](https://github.com/mgravell/protobuf-net), a library build by [Marc Gravell](https://twitter.com/marcgravell), added [Discriminated Union types](https://github.com/mgravell/protobuf-net/blob/59b83356149a6df5027631421e2d6e03c8708725/src/protobuf-net/DiscriminatedUnion.cs) that is capable of addressing it. Let's just take a look at the first of them.

public struct DiscriminatedUnionObject
{
  private readonly int _discriminator;

 // The value typed as Object
 public readonly object Object;

 // Indicates whether the specified discriminator is assigned
 public bool Is(int discriminator) => _discriminator == ~discriminator;

 // Create a new discriminated union value
 public DiscriminatedUnionObject(int discriminator, object value)
 {
  _discriminator = ~discriminator; // avoids issues with default value / 0
  Object = value;
 }
}

Let's walk through all the design choices that have been made here:

1. *DiscriminatedUnionObject* is a struct. It means, that if a class have a field of this type, it will be stored in the object, without additional allocations (you can think of it as inlining the structure, creating a "fat object")

1. It has only one field for storing the value *Object*. (no matter which type is it).
1. It has only one field, called *_discriminator* to store the tag of the field.

If you generated the Wrapper class, it'd have only one field, of the *DiscriminatedUnionObject* type. Once a message of a specified type is set, the discriminator and the value would be written in the union. Simple, and efficient.

### Summing up

Mapping a generic idea, like a discriminated union, into a platform or a language isn't simple. Again, once it's made in an elegant and an efficient way, I truly believe that it's worth to be named as a pearl.
