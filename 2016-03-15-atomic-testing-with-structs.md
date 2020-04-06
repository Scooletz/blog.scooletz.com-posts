---
layout: post
title: "Atomic* - testing with structs"
date: 2016-03-15 11:00
author: scooletz
permalink: /2016/03/15/atomic-testing-with-structs/
nocomments: true
categories: ["RampUp"]
tags: ["RampUpNet", "struct", "value type"]
imported: true
---

In [the last post](http://blog.scooletz.com/2016/03/10/atomic-in-rampup/) I introduced *Atomic** wrappers. They are extremely cheap to use as they're stack allocated (the structs, not the values they point to) and wrap all the needed atomic operations. Having all that in mind, can you come up with any scenario when value types make things harder? Yes, testing.

### Asserting calls

Sometimes ;-) in your tests you need to assert that a specific call was made. The easiest way to do it is to mock/stub/nsubstitute/fake an interface and being given that object, assert recorded calls at the end of the call. Right, but structs aren't interfaces hence they cannot be mocked. Maybe, one could add an interface covering the struct API and instead of calling struct methods, use the interface? That would be a solution, wouldn't it?

### Box & more

Unfortunately that might work. I'm writing unfortunately because of the penalties one would suffer providing this behavior. The penalties are following:

1. boxing - any value type that is cast to the interface is boxed, which means allocation on the heap. We're fighting for no allocations and all so boxing *Atomics* all the time isn't the best idea

1. dispatching interface - there's a cost of calling an interface method. It's connected with the need of getting information what's hidden behind the interface (take a look at a Joe Duffy [blog post](http://joeduffyblog.com/2015/12/19/safe-native-code/)). If you consider struct methods, they are really simple. Just invoke a method under given address.

This isn't a suitable solution.

### Compile more than once

The current approach which I'm not fond of, is based on a conditional compilation and delegating struct methods calls for tests purposes to a separate mock.

[code language="csharp"]
#if NOTESTS
    [Pure]
    public int Read()
    {
        return *_ptr;
    }

#else
    [Pure]
    public int Read()
    {
        return Mocks.AtomicInt.Read((IntPtr)_ptr);
    }
#endif
```

It works, but it's ugly and needs one to run two compilations. Not to mention that the code being tested is not the code one runs. On the other hand, this solution preserves the purity of value types, so for Release mode it's really fast. I'm working on providing a better solution in this [issue](https://github.com/Scooletz/RampUp/issues/6) and will apply it as soon as it's ready.
