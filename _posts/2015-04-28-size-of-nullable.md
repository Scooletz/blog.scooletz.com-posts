---
layout: post
title: "Size of Nullable"
date: 2015-04-28 11:00
author: scooletz
permalink: /2015/04/28/size-of-nullable/
nocomments: true
categories: ["C#"]
tags: ["alignment", "CLR", "struct"]
imported: true
---

What's the size of nullable structure in C#? Does it depends on anything? How fields are held in the memory. Let's consider following cases: bool?, int?, long?, Guid? One way to get the structure size is use of [Marshal.SizeOf(Type)](https://msdn.microsoft.com/en-us/library/vstudio/5s4920fa%28v=vs.100%29.aspx). Unfortunately this method performs a check whether the passed type is generic. Every nullable is generic, hence we cannot use it. If you take a look at the implementation of this method, there is a call to the private method of the Marshal class, named SizeOfHelper. This method does not perform a check and can be easily used to calculate the size of the struct.

Nullable consists of two fields. The first is *hasValue* which answers to the question if the nullable has a value assigned. The other is the value. If a nullable has no value assigned, the value field will held default(T) value. How this members are aligned in the memory, does it depend on anything? What is the offset from the start of the structure of these specific fields?

To answer the two questions above (size and alignment) please take a look at the following table:
<table>
<tr><th>Type</th><th>Size</th><th>hasValue offset</th><th>value offset</th></tr>
<tr><td>bool?<th>8</th><th>0</th><th>4</th></td></tr>
<tr><td>int?<th>8</th><th>0</th><th>4</th></td></tr>
<tr><td>long?<th>16</th><th>0</th><th>8</th></td></tr>
<tr><td>Guid?<th>20</th><th>0</th><th>4</th></td></tr>
</table>
The first two: bool? and int? are easy to come up with. Bool is equal to int, it takes 4 bytes to store one, so the offset of the value is 4.

What about long? Why does it take 16 bytes, not 12? Why the value starts at 8, not 4? That's because of the struct alignment, which CLR performs to enable nice packing up the given struct. In other way, the struct is aligned to the length of the 64bit CPU registries.

The final example with Guid breaks the rule for long. Or maybe not? The struct size is multiplication of 8 bytes, so it's totally ok for CLR to use 24 bytes as it is already aligned.

If you want to do some checks on your own, you may use [the gist I created](https://gist.github.com/Scooletz/9b78bd52451f76ba0eea "Size of nullable").
