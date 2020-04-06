---
layout: post
title: "Rise of the IAsyncStateMachines"
date: 2016-06-13 09:00
author: scooletz
permalink: /2016/06/13/rise-of-the-iasyncstatemachines/
categories: ["C#"]
tags: ["async", "async await"]
imported: true
---

Whenever you use async/await pair, the compiler performs a lot of work creating a class that handles the coordination of code execution. The created (and instantiated) class implements an interface called [IAsyncStateMachine ](https://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.iasyncstatemachine(v=vs.110).aspx)and captures all the needed context to move on with the work. Effectively, any async method using await will generate such an object. You may say that creating objects is cheap, then again I'd say that not creating them at all is even cheaper. Could we skip creation of such an object still providing an asynchronous execution?

### The costs of the async state machine

The first, already mentioned, cost is the allocation of the async state machine. If you take into consideration, that for every async call an object is allocated, then it can get heavy.

The second part is the influence on the stack frame. If you use async/await you will find that stack traces are much bigger now. The calls to methods of the async state machine are in there as well.

### The demise of the IAsyncStateMachines

Let's consider a following example:

[code language="csharp"]
public async Task A()
{
    await B ();
}
```

Or even more complex example below:

[code language="csharp"]
public async Task A()
{
    if (_hasFlag)
    {
        await B1 ();
    }
    else
    {
        await B2 ();
    }
}

```

What can you tell about these A methods? They do not use the result of  the Bs. If they do not use it, maybe awaiting could be done on a higher level? Yes it can. Please take a look at the following example:

[code language="csharp"]
public Task A()
{
    if (_hasFlag)
    {
        return B1 ();
    }
    else
    {
        return B2 ();
    }
}

```

This method is still asynchronous, as asynchronous isn't about using *async await* but about returning a *Task*. Additionally, it does not generate a state machine, which lowers all the costs mentioned above.

Happy asynchronous execution.
