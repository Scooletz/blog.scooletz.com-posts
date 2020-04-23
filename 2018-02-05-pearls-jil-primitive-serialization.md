---
layout: post
title: "Pearls: Jil, serialization of primitives"
date: 2018-02-05 09:55
author: scooletz
permalink: /2018/02/05/pearls-jil-primitive-serialization/
image: /img/2018/02/pearl.png
categories: ["architecture", "design", "pearls"]
tags: ["architecture", "design", "pearls"]
whitebackgroundimage: true
nocomments: true
---

The last *pearl* *of design* that I covered was an implementation for [the discriminated union](http://blog.scooletz.com/2018/01/25/pearls-the-protobufs-discriminated-union/) in the probuf-net library. Now, it's time to move to an area that is less esoteric in terms of the format, but still intriguing in terms of performance. Time to take a look at the fastest JSON serializer available for .NET, [Jil](https://github.com/kevin-montrose/Jil).

### Is it fast?

Jil is crazy fast. As always, the best way to do something fast, is to skip some steps. You can think about skipping doing some computations, to effectively lower the load for the CPU. When speaking about programming in .NET, one additional thing to skip are allocations. And when speaking about allocations and JSON, the one that should come to our minds are strings. I don't mean strings per se, but object that are transformed into them before writing to the actual input.

### A unique case of GUID

GUID in .NET (or Guid) stands for Globally Unique Identifier. It's a structure, 16 bytes long, providing a unique identity generator (let's not deal with comb guids and other types for now). It's frequently used whenever its the client responsibility to generate an id. It's also cheap, as you don't need to call the third party to get a globally unique id.

Let's take a look how Guid is serialized by another library for .NET, probably, the most popular one, JSON.NET

```csharp
public override void WriteValue(Guid value)
{
    InternalWriteValue(JsonToken.String);

    string text = null;

#if HAVE_CHAR_TO_STRING_WITH_CULTURE
    text = value.ToString("D", CultureInfo.InvariantCulture);
#else
    text = value.ToString("D");
#endif

    _writer.Write(_quoteChar);
    _writer.Write(text);
    _writer.Write(_quoteChar);
}
```

As you can see above (and you can see it on [GitHub](https://github.com/JamesNK/Newtonsoft.Json/blob/master/Src/Newtonsoft.Json/JsonTextWriter.cs#L719-L734)), JSON.NET first calls *.ToString*, allocating  32 characters in a form of a string, just to pass it to the *TextWriter* instance. Yes, this instance will end its life shortly after writing it to the writer, but still, it will be allocated. What could be done better?

### Jil to the rescue

We've got a text writer. If we had an additional buffer of *char[]* we could write characters from Guid to the buffer and then pass it to the text writer. Unfortunately, Guid does not provide a method to write its output to the buffer. Jil, provides one instead. It has an additional Guid structure that enables to access internal fields' values of Guid. With this and a buffer, we can write to a text writer without allocating an actual string. The code is much to big to paste it in here. Probably it must be big to be that fast;-) Just follow this link to see [the selected lines](https://github.com/kevin-montrose/Jil/blob/0631e89ca667bccea353f1e7394663bedf91d9f8/Jil/Serialize/Methods.cs#L48-L220).

### Summary

The mentioned optimization for Guid type is just one of many, that you can find in the awesome Jil library. As always, *do not allocate*, is one of the top commandments when talking about performance.
