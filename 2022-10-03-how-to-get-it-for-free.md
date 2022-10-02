---
layout: post
title: "Pearls: Microsoft Orleans Reader and different binary inputs"
date: 2022-10-03 07:00
author: scooletz
permalink: /2022/10/03/pearls-microsoft-orleans-reader-different-binary-inputs
image: /img/2022/orleans.png
categories: ["pearls", "dotnet", "Orleans"]
tags: ["pearls", "dotnet", "Orleans"]
whitebackgroundimage: true
twitter: true
---

Microsoft Orleans is a project that originates from Microsoft Research. It implements a virtual actor model and allows you to write scalable distributed applications. Every distributed system usually has to address a few similar problems like work scheduling or data storage. Beside that, it needs to have a protocol to communicate between the nodes. To have a protocol implemented, it needs to be able to write and read messages in an efficient manner. Let's take a look at some of the serialization aspects of Orleans then and see one of the design pearls.

### Serialization

Orleans are like any other application in a sense, that they create objects and then use them to perform some operations. As vague as it sounds, this is how applications work in .NET (unless, you do some crazy, no alloc magic). If an application requires communication with other processes or nodes, or if it requires writing its state to a storage to later read it from it, it requires some way of transforming in memory state into something that usually in a form of a chunk of bytes. Depending on the approach taken and the case (because sometimes you need to handle both) the intermediate medium will be a `Stream` or a `Memory<byte>` or other representation of chunks of bytes.

The process of hydration, where raw data are read from a storage to be transformed into fully fledged object is called deserialization.

### Serialization in Orleans

In case of Orleans, the aspects of serialization are addressed by a separate project called [Orleans.Serialization](https://github.com/dotnet/orleans/tree/main/src/Orleans.Serialization). If you dive into it, shortly you'll find a general purpose [Serializer](https://github.com/dotnet/orleans/blob/main/src/Orleans.Serialization/Serializer.cs), that handles writing an object representation to and reading it from:

1. `byte[]` - a regular array of bytes, that can be easily wrapped in a `Span<byte>`
1. `Memory<byte>` - a memory, supported either by an array or some memory manager
1. `Stream` - a regular .NET stream
1. `IBufferWriter<byte>` - an abstraction over anything that allows to write chunks of bytes and can support pooling on it own

Let's admit, the number of options here is astonishing! Also, some of them might be quite different in handling. Reading from a `Memory<byte>`, that can be accessed by its `.Span` property, is something different than reading from a `Stream` that requires some additional management. How Orleans handle multiple cases then?

Let's examine the reading part, particularly the case of handling the different types of sources to read from.

### Reader in Orleans

To provide a coherent way of reading _any_ byte source, `Reader<TInput>` is used (declared: [here](https://github.com/dotnet/orleans/blob/cb9faf3532e9efb69e468caa8c791a4c9324d785/src/Orleans.Serialization/Buffers/Reader.cs#L266)). To create a reader, [factory methods](https://github.com/dotnet/orleans/blob/cb9faf3532e9efb69e468caa8c791a4c9324d785/src/Orleans.Serialization/Buffers/Reader.cs#L207-L252) are used. The caller does not need to specify the input on their own, but rather just `Reader.Create(yourInput,....)` to get a reader.

The `Reader<TInput>` itself is a `ref struct`, meaning, that it's:

1. stack allocated
1. cannot be boxed, so...
1. can be used only in a synchronous flow

As `Reader<TInput>` is generic, to create a new one, the type must be closed. If it's specified by a structure, according to the generic rules, the type will be not shared with other structures. Meaning, that if you check something for `<TInput>` you might be sure that it will be preserved. For reference types, this would not work as the implementation is shared between them. This property is often used to write some conditions that are checked once, but still, the majority of the code of the structures is shared.

Let's take a look at the actual longer snippet of code. It's amended with loads of comments to make the comments right next to the code that they discuss. The fields not needed for the reasoning are removed.

```csharp
// As a non-ref struct is needed to specify the type 
// of the generic Reader<TInput>. This is used for Span<byte>.
public readonly struct SpanReaderInput { }

public ref struct Reader<TInput>
{
  // Three checks that are for JIT only.
  // For Span and ReadOnlySequence they will be optimized away.
  readonly static bool IsSpanInput = 
    typeof(TInput) == typeof(SpanReaderInput);
  readonly static bool IsReadOnlySequenceInput = 
    typeof(TInput) == typeof(ReadOnlySequence<byte>);
  readonly static bool IsReaderInput = 
    typeof(ReaderInput).IsAssignableFrom(typeof(TInput));

  // The current span to read from
  private ReadOnlySpan<byte> _currentSpan;
  private SequencePosition _nextSequencePosition;
   private TInput _input;
        
  // force aggressive inlining
  [MethodImpl(MethodImplOptions.AggressiveInlining)]
  internal Reader(TInput input, SerializerSession session, long globalOffset)
  {
    // The condition will be optimized away for ReadOnlySequence<byte>.
    // Only the path below will be in ctor while other branches will be optimized away.
    if (IsReadOnlySequenceInput)
    {
      // The Unsafe.As is used to cast between ref TInput and ref ReadOnlySequence<byte>.
      // In this case we know that it's true, due to the check above, so it's safe do it.
      ref var sequence = ref Unsafe.As<TInput, ReadOnlySequence<byte>>(ref input);

      _input = input;

      // Sequences may contain multiple chunks, position allows to walk through them.
      _nextSequencePosition = sequence.Start;

      // Get the current Span and assign it.
      _currentSpan = sequence.First.Span;
    }
    else if (IsReaderInput)
    {
      // Derived from ReaderInput.
    }
    else
    {
      throw new NotSupportedException();
    }
    Session = session;
  }

  [MethodImpl(MethodImplOptions.AggressiveInlining)]
  internal Reader(ReadOnlySpan<byte> input, SerializerSession session, long globalOffset)
  {
    // The condition will be optimized away for ReadOnlySpan<byte>.
    // Only the path below will be in ctor while the rest will be optimized away.
    if (IsSpanInput)
    {
      // No need to remember the input, it's just an empty unneeded struct.
      _input = default;
      _nextSequencePosition = default;

      // The span itself is assigned to the current span.
      _currentSpan = input; 
    }
    else
    {
        throw new NotSupportedException();
    }
    Session = session;
  }
```

### How to read the Reader then

As you see above, `Reader` is defined as a `ref struct`. It uses an interesting way of leveraging generic argument types to differentiate between different kinds of inputs. It allows to use a shared implementation of one reader. The last but not least, the fact that only one path will be executed for each `Reader<TInput>` makes it even more performant. I strongly encourage you to take a look on your own, how other methods use conditions based on the input type and do remember that _a generic closed with a struct type, is JITted separately_.

### Summary

Microsoft Orleans consists of many interesting parts to deliver an environment and a framework to build distributed apps on top of it. I hope you'll find more pearls like this one above in its source code.