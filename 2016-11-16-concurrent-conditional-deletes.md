---
layout: post
title: "Concurrent conditional deletes"
date: 2016-11-16 09:55
author: scooletz
permalink: /2016/11/16/concurrent-conditional-deletes/
nocomments: true
image: /img/2016/11/stocksnap_hwq3vmryr4.jpg
categories: ["C#"]
tags: ["concurrency", "concurrent c# dotnet"]
imported: true
---

### **TL;DR**

Sometimes **System.Collections.Concurrent** provide not enough methods to squeeze the top performance out of them. Below you can find a small extension method that will make your life easier in some cases

### **Concurrent dictionary**

The concurrent dictionary provides a lot of useful methods. You can *TryAdd*, *AddOrUpdate*. Additionally, you can use *TryUpdate* which provides a very nice method for ensuring optimistic concurrency. Let's take a look at its signature:

public bool TryUpdate(TKey key, TValue newValue, TValue comparisonValue)

It enables to replace a value under a specific key with the *newValue* only if the previous value is equal to *comparisonValue*. It's an extremely powerful API. If you create a new object for a specific key to update it, it enables to replace that object without locks with a single method call. Of course, if the method fails, it returns *false* and it's up to you to retry.

What about deletes? What if I wanted to remove an entry only if nobody changed it's value in the meantime? What if I wanted to have an optimistic concurrency for deletes as well. Is there a method for it? Unfortunately no. The only method for removal is

public bool TryRemove(TKey key, out TValue value)

which removes the value unconditionally returning it. This breaks the optimistic concurrency as we can't ensure that the removed entry wasn't modified. What can be done to make it conditional?

### Explicit interfaces

The ConcurrentDictionary class implements a lot of interfaces, one of them is

ICollection<KeyValuePair<TKey, TValue>>

This interface has one particular method, enabling to remove a pair of values.

ICollection<KeyValuePair<TKey, TValue>>.Remove(KeyValuePair<TKey, TValue> kvp

If you take a look into implementation, it uses a private method of the dictionary to remove the key only if the value is equal to the value of the pair. Now we can write a simple extension method to provide a conditional, optimistically concurrent removal of a key

static class ConcurrentDictionaryExtensions
{
    public static bool TryRemoveConditionally<TKey, TValue>(
        this ConcurrentDictionary<TKey, TValue> dictionary, TKey key, TValue previousValueToCompare)
    {
        var collection = (ICollection<KeyValuePair<TKey, TValue>>)dictionary;
        var toRemove = new KeyValuePair<TKey, TValue>(key, previousValueToCompare);
        return collection.Remove(toRemove);
    }
}

which closes the gap and makes the API of the concurrent dictionary support all the operations under optimistic concurrency.

### Summary

With this simple tweak you can use a concurrent dictionary as a collection that supports fully an optimistic concurrency.
