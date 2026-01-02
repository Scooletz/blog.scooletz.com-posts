---
layout: post
title: "It's beginning to look a lot like vectors"
date: 2025-12-31 06:00
author: scooletz
permalink: /2025/12/31/look-a-lot-like-vectors
image: /img/2025/look-like-vectors.webp
categories: ["performance", "vectors", "SIMD"]
tags: ["performance", "vectors", "SIMD"]
whitebackgroundimage: true
twitter: false
---

To enjoy this post the most, please play _It's Beginning to Look a Lot Like Christmas_ by Michael Bublé.

---

It's beginning to look a lot like vectors,<br>
Everywhere you go.<br>
Take a look at your loops and then, they're faster once again,<br>
With `SIMD` lanes and hardware paths aglow.<br>

It's beginning to look a lot like vectors,<br>
In [`System.Numerics`](https://learn.microsoft.com/en-us/dotnet/api/system.numerics) store.<br>
But the prettiest sight to see is the hardware flag so free,<br>
[`IsHardwareAccelerated`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector256.ishardwareaccelerated) on your core.<br>

A pair of [`Vector128`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector128)s and [`Vector256`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector256)s that fits,<br>
Is the wish of every dev.<br>
For `ARM`, a [`Vector64`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector64) will do, making loops feel fresh and new,<br>
A wish for `JIT` and Jen.<br>

With `Add`, `Subtract`, and `Multiply`, the data starts to fly,<br>
No scalar path can quite compare to them.<br>
And a [PR for xxHash](https://github.com/ravendb/ravendb/pull/21267), showed us all the way to dash,<br>
And optimize our code right to the stem.<br>

It's beginning to look a lot like vectors,<br>
So check the buffer's size.<br>
With a scalar path for tails, your vectorized code prevails,<br>
Call [`LoadAligned`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector256.loadaligned) and [`LoadUnsafe`](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.intrinsics.vector256.loadunsafe) wise.<br>

