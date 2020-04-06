---
layout: post
title: ".NET volatile write performance degradation in x86"
date: 2016-07-25 09:00
author: scooletz
permalink: /2016/07/25/net-volatile-write-performance-degradation-in-x86/
categories: ["C#", "Optimization"]
tags: ["memory barrier", "optimization", "Volatile"]
imported: true
---

**TL;DR**

This is a summary of my investigation about writing a fast and well designed concurrent queue for akka.net which performance was drastically low for 32bit application. Take a look at [PR here](https://github.com/akkadotnet/akka.net/pull/2200). If you're interested in writing a well performing no-alloc applications with mechanical symapthy in mind or you're simply interested in good .NET concurrency this post is for you.

### Akka.NET

Akka.NET is an actor system's implementation for .NET platform. It has been ported from Java. Recently I spent some time playing with it and reading through the codebase. One of the classes that I took a look into was *UnboundedMailboxQueue* using a general purpose [.NET BCL's concurrent queue](https://github.com/akkadotnet/akka.net/blob/dev/src/core/Akka/Dispatch/MessageQueues/UnboundedMailboxQueue.cs#L12). It looked strange to me, as knowing a structure Envelope that is passed through this queue one could implement a better queue. I did it in [this PR](https://github.com/akkadotnet/akka.net/pull/2200) lowering number of allocations by 10% and speeding up the queue by ~8%. Taking into consideration that queues are foundations of akka actors, this result was quite promising. I used the benchmark tests provided with the platform and it looked good. Fortunately Jeff Cyr run some [tests on x86](https://github.com/akkadotnet/akka.net/pull/2200#issuecomment-233641929) providing results that were disturbing. On x86 the new queue was *underperforming*. Finally I closed the PR without providing this change.

### The queue design

The custom queue provided by use a similar design to the original concurrent queue. The difference was using Envelope fields (there are two: message & sender) to mark message as published without using [the concurrent queue state array](http://referencesource.microsoft.com/#mscorlib/system/Collections/Concurrent/ConcurrentQueue.cs,660). Again, knowing the structure you want to passed to the other side via a concurrent queue was vital for this design. You can't make a universal general collection. Note 'general', not 'generic'.

### Volatile

To make the change finally visible to a queue's consumer, *Volatile.Write* was used. The only difference was the type being written. In the BCL's concurrent queue that was bool in an array. In my case it was an object. Both used different overloads of *Volatile.Write(ref ....)*. For sake of reference, Volatile.Write ensures *release barrier* so if a queue's consumer reads status with *Volatile.Read* (the aquire barrier), it will finally see the written value.

### Some kind of reproduction

To know how .net is performing this operations I've used two types and run a sample application with x64 and x86. Let's take a look at the code first.

```csharp
struct VolatileInt
{
int _value;

public void Write(int value)
{
_value = value;
}

public void WriteVolatile(int value)
{
Volatile.Write(ref _value, value);
}
}

struct VolatileObject
{
object _value;

public void Write(object value)
{
_value = value;
}

public void WriteVolatile(object value)
{
Volatile.Write(ref _value, value);
}
}
```

It's really nothing fancy. These two either write the value ensuring *release fence* or just write the value.

### Windbg for x86

The methods had been prepared using *RuntimeHelpers.PrepareMethod()*. A Windbg instance was attached to the process. I loaded sos clr and took a look at method tables of these two types. Because methods had been prepared, they were jitted so I could easily take a look at the jitted assembler. Because x64 was performing well, let's take a look at x86. At the beginning let's check the non-object method, *VolatileInt.VolatileWrite*

[code]
cmp     byte ptr [ecx],al
mov     dword ptr [ecx],edx
ret
```

Nothing heavy here. Effectively, just move a memory and return. Let's take a look at writing the object with *VolatileObject.VolatileWrite*

[code]
cmp     byte ptr [ecx],al
lea     edx,[ecx]
call    clr!JIT_CheckedWriteBarrierEAX
```

Wow! Beside moving some data an additional method is called. The method name is *JIT_CheckedWriteBarrierEAX* (you probably this now that there may be a group of *JIT_CheckedWriteBarrier* methods). What is it and why does it appear only in x86?

### CoreCLR to the rescue

Take a look [at the following snippet](https://github.com/dotnet/coreclr/blob/master/src/inc/jithelpers.h#L347-L375) and compare blocks for x86 and non-x86? What can you see? For x86 there are additional fragments, including the one mentioned before *JIT_CheckedWriteBarrierEAX*. What does it do? Let's take a look at another piece of CoreCLR [here](https://github.com/dotnet/coreclr/blob/master/src/vm/i386/jithelp.asm#L149). Let's not dive into this implementation right now and what is checked during this call, but just taking a look at first instructions of this method one can tell that it'll cost more than the simple int operation

[code title="JIT_CheckedWriteBarrierEAX"]
cmp edx,dword ptr [clr!g_lowest_address]
jb      clr!JIT_CheckedWriteBarrierEAX+0x35
cmp     edx,dword ptr [clr!g_highest_address]
jae     clr!JIT_CheckedWriteBarrierEAX+0x35
mov     dword ptr [edx],eax
cmp     eax,dword ptr [clr!g_ephemeral_low]
jb      clr!JIT_CheckedWriteBarrierEAX+0x37
cmp     eax,dword ptr [clr!g_ephemeral_high]
jae     clr!JIT_CheckedWriteBarrierEAX+0x37
shr     edx,0Ah
```

**Summing up** If you want to write well performing code and truly want to support *AnyCPU,* a proper benchmark tests run with different architectures should be provided.

Sometimes, a gain in one will cost you a lot in another. Even if for now this PR didn't make it, this was an interesting journey and an extreme learning experience. There's nothing better that to answer a childish 'why?' on your own.
