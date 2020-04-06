---
layout: post
title: "Cache me this, cache me that"
date: 2011-01-03 20:00
author: scooletz
permalink: /2011/01/03/cache-me-this-cache-me-that/
nocomments: true
categories: ["Enterprise library", "Optimization", "Profiling"]
tags: ["caching", "optimization", "performance", "profiling"]
imported: true
---

Recently I've been optimizing one application. After a few runs of profile_analyzeData_fixBottlenecks I stopped when an app written with NHibernate intensively using CacheManager from EntLib was spending 5% of their time getting data from cache. As the performance was increased by an order of magnitude, the time spend within _GetData_ method was much to large. I reviewed code and wrote a few statements describing what application require from a cache:

* no expiration and
* it is infrequently erased so
* has big read:write ratio
* the number of cached items can be easly estimated, hence
* the number of cached items will not frequently exceed the max number of item
* once the scavenging occurs (current number > max number of items), it can take a while, since it has a low probability to hit the upper limit of cache items number

Following these points, the performance results of GetData method and need of tuning the app (it was the most obvious point for optimization) I came up with the following *Cache Manager* implementation. The code is simple, oriented for frequent reads (slim rw lock), with all operations having cost of O(1). Adding, when a maximum number of cache items is exceeded takes O(n) because scavenging takes O(n) to clean up cache (the FIFO is used for removing these entries). The removal of key is handled by marking it as removed in the key look up list.

The whole implementation with tests took me 40 minutes, which with other small optimizations improved performance by 10%. The main idea behind this is to give you some thoughts about using the external *all in one* solutions versus writing a small, custom one, for only your needs. Hope the code example help as well :-)

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Threading;
using Trader.Core.Exceptions;
using Trader.Infrastructure.IoC.Wcf;

namespace Trader.Infrastructure.Caching
{
    ///
    /// The implementation of ,
    /// using fifo strategy for scavenging to many objects in a cache.
    ///
    ///
    /// This implementation is thread safe and its instances can be shared
    /// by different threads.
    ///
    public class CacheManager : ICacheManager, IDisposable
    {
        private readonly Dictionary _cache;
        private readonly int _capacity;
        private readonly int _maxNumberOfItems;
        private readonly int _numberToRemove;
        private readonly ReaderWriterLockSlim _rwl = new ReaderWriterLockSlim();
        private List _keys;

        public CacheManager(string name, int maximumElementsInCacheBeforeScavenging, int numberToRemove)
        {
            if (maximumElementsInCacheBeforeScavenging < 1)
            {
                throw new ArgumentException("Cannot be less than 1", "maximumElementsInCacheBeforeScavenging");
            }
            if (numberToRemove < 1)             {                 throw new ArgumentException("Cannot be less than 1", "numberToRemove");             }             if (numberToRemove > maximumElementsInCacheBeforeScavenging)
            {
                throw new ArgumentException("Scavenging more than max number of elements in cache", "numberToRemove");
            }

            Name = name;
            _maxNumberOfItems = maximumElementsInCacheBeforeScavenging;
            _numberToRemove = numberToRemove;

            _capacity = maximumElementsInCacheBeforeScavenging + 1;
            _cache = new Dictionary(_capacity);
            _keys = new List(_capacity);
        }

        public string Name { get; private set; }

        public int Count
        {
            get
            {
                using (_rwl.Readlock())
                {
                    return _cache.Count;
                }
            }
        }

        ///
        /// Adds the specified key and value to the cache.
        ///
        /// The key of cache entry.
        /// The value of cache entry.
        ///
        /// This is O(1) operation, but when a number of cache items exceeded,
        /// it calls  method. See more .
        ///
        public void Add(string key, object value)
        {
            key.ThrowIfNull("key");
            using (_rwl.WriteLock())
            {
                CacheItem item;
                if (_cache.TryGetValue(key, out item))
                {
                    // if exists only update
                    item.Value = value;
                }
                else
                {
                    item = new CacheItem(value, _keys.Count, key);
                    _cache[key] = item;
                    _keys.Add(item);

                    // scavange if needed
                    if (_cache.Count > _maxNumberOfItems)
                    {
                        Scavange();
                    }
                }
            }
        }

        public bool Contains(string key)
        {
            using (_rwl.Readlock())
            {
                return _cache.ContainsKey(key);
            }
        }

        public void Flush()
        {
            using (_rwl.WriteLock())
            {
                _cache.Clear();
                _keys.Clear();
            }
        }

        public object GetData(string key)
        {
            using (_rwl.Readlock())
            {
                CacheItem result;
                _cache.TryGetValue(key, out result);

                if (result != null)
                {
                    return result.Value;
                }
                return null;
            }
        }

        ///
        /// Removes a specified key from cache.
        ///
        /// The key to be removed.
        ///
        /// When non existing key passed, simply does nothing.
        /// This is O(1) operation.
        ///
        public void Remove(string key)
        {
            using (_rwl.WriteLock())
            {
                CacheItem result;
                _cache.TryGetValue(key, out result);

                if (result != null)
                {
                    // mark key as removed for scavenging optimization
                    _keys[result.KeyIndex] = null;
                    _cache.Remove(key);
                }
            }
        }

        public void Dispose()
        {
            _rwl.Dispose();
        }

        ///
        /// Scavanges elements, deleting at most n entries, where n is numberToRemove
        /// from ctor parameter.
        ///
        /// A number of scavanged items.
        ///
        /// This method has O(n) complexity. It iterates and recreates index list.
        ///
        public int Scavange()
        {
            var removedSoFar = 0;
            for (var i = 0; i < _keys.Count && removedSoFar < _numberToRemove; i++)
            {
                if (_keys[i] != null)
                {
                    _cache.Remove(_keys[i].Key);
                    _keys[i] = null;
                    ++removedSoFar;
                }
            }

            var newKeys = new List(_capacity);
            for (var i = 0; i < _keys.Count; i++)
            {
                if (_keys[i] != null)
                {
                    newKeys.Add(_keys[i]);
                }
            }

            // fix indexes
            for (var i = 0; i < newKeys.Count; i++)
            {
                newKeys[i].KeyIndex = i;
            }

            _keys = newKeys;

            return removedSoFar;
        }

        #region Nested type: CacheItem

        [DebuggerDisplay("Cache item: Key {Key}, KeyIndex {KeyIndex}")]
        private class CacheItem
        {
            public readonly string Key;
            public object Value;
            public int KeyIndex;

            public CacheItem(object value, int keyIndex, string key)
            {
                Key = key;
                Value = value;
                KeyIndex = keyIndex;
            }
        }

        #endregion
    }
}
```
