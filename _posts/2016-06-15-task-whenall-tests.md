---
layout: post
title: "Task.WhenAll tests"
date: 2016-06-15 09:00
author: scooletz
permalink: /2016/06/15/task-whenall-tests/
nocomments: true
categories: ["C#"]
tags: ["async", "async await"]
imported: true
---

In [the last post](https://blog.scooletz.com/2016/06/13/rise-of-the-iasyncstatemachines/) I've shown some optimizations one can apply to reduce the overhead on creating asynchronous state machines. Let's dive into the async world again and consider helper methods provided by the Task class, especially [Task.WhenAll](https://msdn.microsoft.com/en-us/library/hh160374(v=vs.110).aspx).

```csharp
public static Task WhenAll(params Task[] tasks)
```

The method works in a following way. It accepts an array of tasks and returns a task that will finish as soon as all of the underlying tasks are finished. Applying this method in some scenarios may provide an improvement gain, as one can run a few tasks in parallel. It has a drawback though.

Let's consider following code

```csharp
public Task<int> A()
{
  await B1();
  await B2();
  return C ();
}
```

If *b1* and *b2* could be executed in parallel (for instance, they access Azure Table Storage), this method could be rewritten in a following way.

```csharp

public Task<int> A()
{
  await Task.WhenAll(B1(), B2());
  return C ();
}

```

What, beside the mentioned performance improvement, changed? Now, method A is no longer a method A. There are two methods which can be randomly executed. One running operations in the following order: B1, B2, C, and the other: B2, B1, C. This effectively means, that your previous test coverage is no longer true. If you want to truly test it, you need to provide suites that will order these B* calls properly and ensure that all permutations will be emitted and tested. Sometimes it has no meaning, sometimes it has. Let's consider a following scenario:
* Two callers are calling A in the same time
* Every B method removes a specific file, failing if it does not exist
* At least one of the callers should succeed

In the first version, it was a pure race for being the first. The first that goes through B1, B2, C would execute properly. Now consider the second version of A with two callers executing following operations in the specified order:

* Caller 1: B1, B2, C
* Caller 2: B2, B1, C

As you can see, it's a typical deadlock scenario and both callers would fail.

As always, there's no silver bullet and if you want to use Task.WhenAll to speed up your application by running operations in parallel, you must embrace the fact of a possibly non linear execution.

Happy awaiting.
