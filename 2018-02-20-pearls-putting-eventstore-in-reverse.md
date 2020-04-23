---
layout: post
title: "Pearls: putting EventStore in reverse"
date: 2018-02-20 09:55
author: scooletz
permalink: /2018/02/20/pearls-putting-eventstore-in-reverse/
image: /img/2018/02/pearl.png
categories: ["architecture", "design", "pearls"]
tags: ["architecture", "design", "pearls"]
whitebackgroundimage: true
nocomments: true
---

Pearls of design, beautiful patterns, efficient approaches. After covering [Jil](http://blog.scooletz.com/2018/02/05/pearls-jil-primitive-serialization/) and its extremely efficient serialization of primitives it's time to put things in reverse. By things, in this case, I mean [EventStore](http://blog.scooletz.com/2014/06/11/pearls-eventstore-transaction-log/), the event centric database that I already presented once. Now, it's time to visit its ability to move back in time and traverse its log in the opposite direction.

### Log all the things

EventStore uses a traditional log mechanism for storing its data in a transactional way. The mechanism can be described as a never-ending log file, that has new data appended at the end of itself. Of course different databases implement the "never ending file" in different ways, still, the logical approach is the same: whenever data are updated/inserted we log these operations to a file.

Appending operations has an interesting property. Whenever other components break, whenever hardware restarts, the log is the single source of truth that can be traversed and reapplied to all the components that didn't catch up before failure. How to make sure that a specific piece of log was written properly though? That the data what we stored, are the data that survived the crush?

### Make it twice

Below you can see a piece of code that is extracted from the writer:

```csharp
var workItem = _writerWorkItem;
var buffer = workItem.Buffer;
var bufferWriter = workItem.BufferWriter;

buffer.SetLength(4);
buffer.Position = 4;
record.WriteTo(bufferWriter);
var length = (int) buffer.Length - 4;
bufferWriter.Write(length); // length suffix
buffer.Position = 0;
bufferWriter.Write(length); // length prefix
```

We can see the following:

1. there's a buffer, which is a regular *MemoryStream*
1. its position is set to 4, leaving 4 initial bytes empty
1. the whole *record* is written to the buffer

1. its length is calculated
1. the length is written at the beginning and and at the end

This looks like storing 4 bytes too much. On the other hand, it's a simple check mechanism, used internally by EventStore to check the consistency of the log. If these two values do not match, *something is seriously wrong in chunk* (an original comment from the code). Could the same length written twice be used for something more?

### Put it in reverse Terry

EventStore has additional indexing capabilities, that allow it to move back and forth pretty fast. What if we had none and still wanted to travel through the log?

Consider the following scenario. You use a log approach for you service. Then, for any reason you need to read 10th entry from the end. Having a regular log, you'd need to do the following:

1. Read the log from the beginning counting items till it's done. Sorry, there's no other point and it's painfully blunt method for doing it.

If we have the length written twice though, what you can do is to read 4 last bytes of the log, it will always be length and **move backward** to the previous entry. The number of bytes to move backward?

```csharp
var moveBackBy = length + 2 * sizeof(int)
```

as two lengths are written on a single integer.

With this, you don't need any additional index. The log itself is sufficient enough to move forward and backward.

### Summary

Data structures and data format nowadays are not popular topics. As you saw above, adding just one integer added a lot of capabilities to a simple append only file. Next time, before "just using" another data store, think about your data and the format you're gonna use. It can make a real difference.
