---
layout: post
title: "Improving Nethermind performance"
date: 2020-11-23 11:00
author: scooletz
permalink: /2020/11/23/improving-Nethermind-performance
image: /img/2020/11/Ethereum.png
categories: ["performance", "dotnet", "ethereum", "NethermindEth"]
tags: ["performance", "dotnet", "ethereum", "NethermindEth"]
whitebackgroundimage: true
---

If you had to choose, would you like to use a slower Ethereum node or a faster one? I'd go with the fast. It'd be foolish to not want more for less! Recently, I spent some time on optimizing one of the Ethereum clients, [Nethermind](https://nethermind.io). It's written in .NET Core and provides an amazing opportunity for .NET engineers to work with a product that has a large active codebase. From the Ethereum point of view, it must be fast as well to do its operations efficiently. This post discusses various improvements and optimizations introduced to the codebase.

> NOTE: I'm not an Ethereum expert and any mistakes made in this post should be solely attributed to me. Nethermind team did it best to share knowledge with me, but being given my limited involvement in Ethereum, I focused on the performance aspects, not on the Ethereum itself.

### Minor things first

The initial work was related to some limitations in JSON RPC handling. Nethermind nodes allow to enable JSON RPC communication and in some cases a node could misbehave, handling requests in a bit longer periods of time. The initial work was based on a quick review and going through my usual checklist of things:

- `async over sync` and `sync over async`
- `Encoding`, especially `Encoding.Utf8`
- possible spanification

This resulted in [this PR](https://github.com/NethermindEth/nethermind/pull/2435) which changed the pool of heavy objects to be async friendly and a single removal of allocation. There were more opportunities for improvements, but again, trying to be as effective as possible only quick things were changed and applied. Later on [a separate PR](https://github.com/NethermindEth/nethermind/pull/2490) changed the usage of `ConcurrentStack` to prefer `.TryPop` over `.Count` which needs to traverse the whole stack. This change was not benchmarked, but as we don't count any longer, it must be better ;)

### Too Big to Pool

The next PR that [introduced LargeArrayPool](https://github.com/NethermindEth/nethermind/pull/2493) was based on much heavier profiling. The setup took me a while as I needed to setup an `xDai` node ([xDai STAKE](https://www.xdaichain.com)), fully synchronize it and then run it locally with an additional app using JSON RPC. During profiling, in one of the app setup, I noticed a huge spike of allocations followed up by a CPU usage increase. After analyzing snapshots, I noticed that the path responsible for the surge was the line from `ArrayPool<byte>.Shared` shown below:

```csharp
// The request was for a size too large for the pool.  Allocate an array of exactly the requested length.
// When it's returned to the pool, we'll simply throw it away.
buffer = GC.AllocateUninitializedArray<T>(minimumLength);
```

The shared instance of `ArrayPool<>`, [TlsOverPerCoreLockedStacksArrayPool](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Private.CoreLib/src/System/Buffers/TlsOverPerCoreLockedStacksArrayPool.cs#L141-L143) by default provides buffers up to `1MB`. Requested buffers bigger than that are allocated and discarded. In my case, a specific Ethereum Virtual Machine call required much more memory. As this case is highly unlikely and it could be wasteful to provide segments for `2MB`, `4MB` etc., I provided as simple `LargerArrayPool` that works in a following way:

```csharp
public override byte[] Rent(int minimumLength)
{
  // the usual, standard case
  if (minimumLength <= _arrayPoolLimit)
  {
    return _smallPool.Rent(minimumLength);
  }

  // 1MB - 8MB, just use one size of buffers
  if (minimumLength <= _largeBufferSize)
  {
    return RentLarge();
  }

  // any other case, just allocate
  return new byte[minimumLength];
}
```

This would address the occasional heavy requests. The estimated size of the pool was based on the limited pool of JSON RPC processors and is correlated with number of CPUs. In modern environments, it's common to observe a correlation between number of cores with RAM (for example: [SKU in Azure](https://docs.microsoft.com/en-us/partner-center/develop/product-resources#sku)). Introducing correlation made sense for the majority of cases.

After having conversation with the Nethermind team we coined the phrase `Too Big to Pool` which was summarized in their communicator as follows:

> Too Big to Pool, a new drama about the allocatey nature of our consumption-based lives

I think this would make a really good PR title after all.

### Potentially cheaper array based LRU cache

The very last PR, named [Potentially cheaper array based LRU cache](https://github.com/NethermindEth/nethermind/pull/2497), was related to the observation made when profiling for the buffers' allocation. When reviewing memory snapshots, I noticed that there's a lot of small objects (tens or hundreds of thousands) being allocated. The reason for this was the implementation of the LRU (Last Recently Used) cache used internally by Nethermind, to memoize some of the data and calculations. Let's take a look how `LruCache` structured its data before the PR

```csharp
public class LruCache<TKey, TValue> : ICache<TKey, TValue>
{
  private readonly int _maxCapacity;
  private readonly Dictionary<TKey, LinkedListNode<LruCacheItem>> _cacheMap;
  private readonly LinkedList<LruCacheItem> _lruList;

  private class LruCacheItem
  {
    public LruCacheItem(TKey k, TValue v)
    {
      Key = k;
      Value = v;
    }

    public TKey Key;
    public TValue Value;
  }
```

`LruCache` internally used a dictionary for keeping the mapping between a key and the value. To enable `LRU` behavior, values with keys were stored as `LruCacheItem` in `LinkedListNode`. This allow to easy find the last item that needs to be removed and to find its key, that needs to be removed from `_cacheMap` as well. All the methods were synchronized, so there was no concurrency related concerns in here. Let's dive a bit deeper how the `LinkedListNode<T>` looks like ([original code](https://github.com/dotnet/runtime/blob/e2312e1edb97d51ea0feded221f67506a1ce67a6/src/libraries/System.Collections/src/System/Collections/Generic/LinkedList.cs#L612-L660)).

```csharp
public sealed class LinkedListNode<T>
{
  internal LinkedList<T>? list;
  internal LinkedListNode<T>? next;
  internal LinkedListNode<T>? prev;
  internal T item;
}
```

The node itself is a class, so it has an object header that is put in memory before the fields. An object header is `8 bytes` long for 32bits and `16 bytes` long for 64bits. Then, there are 4 references in here so this make it as follows

1. object header
1. list (a reference)
1. next (a reference)
1. prev (a reference)
1. item (a reference because `LruCacheItem` is a class)

Let's discuss the structure at `LruCacheItem` then. Depending on `TKey` and `TValue` it may have a different size, so let's focus on its own memory consumption

1. object header

All of the above could be summarized in the following table that presents an overhead for a single key value pair stored in this structure (without the size of the key and the value themselves):

|  Item | Description  | 32-bit | 64-bit  |
|---|---|---|---|
| LinkedListNode | class overhead (object header)  |  8  | 16  |
| LinkedListNode.prev |  an object reference  | 4  | 8  |
| LinkedListNode.next  |  an object reference  | 4  | 8  |
| LinkedListNode.list  | an object reference  | 4  | 8  |
| LruCacheItem  | class overhead (object header)  | 8  | 16  |
|  **Total** |   | **24 bytes**  | **48 bytes** |

One could argue if 24 bytes or 48 bytes per a key-value pair is good or not, but if we take into consideration that Nethermind can cache a million of entries, this can make it more significant. After all why one would like to allocate additional **48MB**?

My initial spike that resulted in [the full blown PR](https://github.com/NethermindEth/nethermind/pull/2497), was based on introduction of an **array of structs**. A `struct` in .NET, beside being a foundation for the usual question during recruitment process, has some advantages. As it's not an object, it does not have an object header! This meant that if I could replace both, the `LinkedListNode` and the `LruCacheItem` and replace them with a `struct` in an array, I could save 16/32 bytes per one entry! I gave it a go and I reimplemented the linked list using `int`s instead of references, that would point to the same array. With this approach the implementation changed to be based on the following:

```csharp
public class LruCache<TKey, TValue> : ICache<TKey, TValue> where TKey : notnull
{
  private readonly int _maxCapacity;
  private readonly Dictionary<TKey, int> _cacheMap;
  private Node[] _list;

  struct Node
  {
      public const int Null = -1;

      public int Prev;
      public int Next;
      public TValue Value;
      public TKey Key;
  }
```

and resulted in the following estimated size:

|  Item | Description  | 32-bit | 64-bit  |
|---|---|---|---|
| Cell.prev | an int32  |  4  | 4  |
| Cell.next | an int32  |  4  | 4  |
|  **Sum** |   | **8 bytes** | **8 bytes** |

Which gave the final difference:

|  Approach | 32-bit | 64-bit  |
|---|---|---|
| LinkedList |  24  | 48  |
| Array of structs |  8 | 8 |
| **Saved bytes per entry** |  **16** | **40** |

This means that caching 1 million of objects in 64bit environment should take **40MB less**! A positive side effect was observed in the response time of the cache, which was reduced by ~20%. I attribute it to  to the limited number of hops between objects but didn't spend time on digging in assembly.

### Summary

This was a very intensive period of time, that resulted in a several PRs that hopefully will make Nethermind a bit faster and a bit more stable. I'd like to send **big kudos to Nethermind team**. This is a truly great, open team, working on a really interesting project. Keep on rocking [Nethermind](https://nethermind.io)!
