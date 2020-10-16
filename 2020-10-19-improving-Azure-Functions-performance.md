---
layout: post
title: "Improving Azure Functions performance"
date: 2020-10-19 11:00
author: scooletz
permalink: /2020/10/19/improving-Azure-Functions-performance
image: /img/2020/10/azure-functions.png
categories: ["Azure", "AzureFunctions", "serverless", "performance", "dotnet"]
tags: ["Azure", "AzureFunctions", "serverless", "performance", "dotnet"]
whitebackgroundimage: true
---

There are environments where a performance and a speed of execution might not be not that important. It might be an extremely powerful server that can host anything and isn't used right now. It might be also a goal to spend as much money as you can and perform a **Denial of Wallet** on your company, especially, if you work in a Pay as You Go environment. There are cases where you want to minimize the cost though. For instance, running your Azure Functions in a more effective way, can save a lot of precious GB-s  allowing you, to use the free grant much more effectively.

In last few days I spent some time to save some money for our (mine and yours) Azure Function apps and provides some improvements in the area of performance. This post discusses this few improvements.

### Allocating less memory when throwing an exception

[The first PR](https://github.com/Azure/azure-webjobs-sdk/pull/2510) was focused on the way exceptions are handled and thrown. I found that the `ExceptionFormatter` was quite allocatey and I followed it up with a measurement. The initial intuition was based on the fact, that internal methods used for the building process were not accepting a `StringBuilder` but rather were returning a `string`. This resulted in multiple concatenations which are costly, because executed multiple times can have a cumulative characteristics. Let's take a look at an example method before the change

```csharp
private static string GetStackForAggregateException(Exception exception, AggregateException aggregate)
{
  var text = GetStackForException(exception, includeMessageOnly: true);
  for (int i = 0; i < aggregate.InnerExceptions.Count; i++)
  {
    text = $"{text}{Environment.NewLine}---> (Inner Exception #{i}) {GetFormattedException(aggregate.InnerExceptions[i])}<---{Environment.NewLine}";
  }

  return text;
}
```

As you can see, the method is `string` returning and uses formatting. Of course on a happy path it may iterate only once. Still if you count up all the formatting, you'll see a potentially cumulative nature of this kind of flow.

The propose and successfully merged solution was based on applying `StringBuilder` and pushing it down through many methods. This changed the API from returning `string` to accepting an additional `StringBuilder` parameter. Let's take a look at the method after the change was applied.

```csharp
private static void GetStackForAggregateException(StringBuilder sb, Exception exception,
    AggregateException aggregate)
{
  GetStackForException(sb, exception, includeMessageOnly: true);
  for (int i = 0; i < aggregate.InnerExceptions.Count; i++)
  {
    sb.AppendLine();
    sb.Append($"---> (Inner Exception #{i}) ");
    GetFormattedException(sb, aggregate.InnerExceptions[i]);
    sb.AppendLine("<---");
  }
}
```

As you can see, instead of formatting the text again and again, a usage of `.Append` was sufficient to reduce the memory footprint greatly. There are still some allocations in there, as no fancy caching was introduced in the PR, but removing a few kilobytes of allocations per exception being thrown is not that bad.

### SharedQueueWatcher redesigned

[The second PR](https://github.com/Azure/azure-webjobs-sdk/pull/2593) was related to the `SharedQueueWatcher` component. The `SharedQueueWatcher` is a part responsible for notifying a function that is triggered via a `Azure Storage Queue` message that the wait is over and there's a message that can be picked up. The waiting is introduced to do not ping the queue over and over again. For queues that are  mostly empty it can span longer periods of time, so being able to give a function the kick is a really nice thing.

This changed was focused on changing the behavior on the hot path. Any notification mechanism consist of registering notifications and invoking them. Initially in this case a `ConcurrentDictionary<string,ConcurrentBag<INotificationCommand>>` was used for the implementation

```csharp
internal class SharedQueueWatcher : IMessageEnqueuedWatcher
{
  private readonly ConcurrentDictionary<string, ConcurrentBag<INotificationCommand>> _registrations =
      new ConcurrentDictionary<string, ConcurrentBag<INotificationCommand>>();

  public void Notify(string enqueuedInQueueName)
  {
    if (_registrations.TryGetValue(enqueuedInQueueName, out var queueRegistrations))
    {
      // hot path a bit slow, materializes the array every time
      foreach (var registration in queueRegistrations.ToArray())
      {
          registration.Notify();
      }
    }
  }

  public void Register(string queueName, INotificationCommand notification)
  {
    _registrations.AddOrUpdate(queueName,
      new ConcurrentBag<INotificationCommand>(new INotificationCommand[] { notification }),
      (i, existing) => { existing.Add(notification); return existing; });
  }
}
```

This ensured that the `.Register` can be done relatively fast, but `Notify` required calling `.ToArray` and materializing registrations from the `ConcurrentBag` (to see why it's not the best idea I encourage to visit and join [Async Expert](https://asyncexpert.com)). If we take into consideration an intuitive approach that the hot `.Notify` path should be faster we could redesign this component into the one, that does nothing when notifying and some additional work when registering.

```csharp
internal class SharedQueueWatcher : IMessageEnqueuedWatcher
{
  private readonly ConcurrentDictionary<string, INotificationCommand[]> _registrations =
    new ConcurrentDictionary<string, INotificationCommand[]>();

  public void Notify(string enqueuedInQueueName)
  {
    if (_registrations.TryGetValue(enqueuedInQueueName, out var queueRegistrations))
    {
      // hot path is fast, just accesses the array
      foreach (var registration in queueRegistrations)
      {
        registration.Notify();
      }
    }
  }

  public void Register(string queueName, INotificationCommand notification)
  {
    _registrations.AddOrUpdate(queueName, new[] { notification },
      (i, existing) =>
      {
        var updated = new INotificationCommand[existing.Length + 1];
        existing.CopyTo(updated, 0);
        updated[existing.Length] = notification;
        return updated;
      });
  }
}
```

### Lighter QueueCausalityManager for Azure Storage Queues

[The last PR](https://github.com/Azure/azure-webjobs-sdk/pull/2599) was about making a JSON operation lighter. Nowadays it's much easier to serialize and deserialize JSON faster. All you need to do is to use `System.Text.Json` and enjoy the speed. Azure Functions project uses the `Newtonsoft.Json` though. One could start arguing for the big rewrite, but even with `Newtonsoft` there are fast APIs that you could use. One of the mentioned APIs is `JsonTextReader`. This component allows you to write code that scans through an object without actually deserializing it. It works as a great substitute when working with data in a schemaless work, just using `JObject` and the rest of `J*` family.

In the case of `QueueCausalityManager` it was all about finding one property that is named `$AzureWebJobsParentId`. This property represents a correlation identifier used for... correlating executions. Because `Azure Storage Queues` do not support headers, adding an external property is one of the ways that you could use for passing a context related values.

Long story short, the implementation was changed from deserializing the whole message to `J*` objects to the following snippet that just scans it to find the value

```csharp
using (var stringReader = new StringReader(text))
{
  using (var reader = JsonSerialization.CreateJsonTextReader(stringReader))
  {
    while (reader.Read())
    {
      if (reader.TokenType == JsonToken.PropertyName && reader.Value.ToString() == ParentGuidFieldName)
      {
        if (reader.Read())
        {
          if (reader.TokenType == JsonToken.String)
          {
            if (Guid.TryParse(reader.Value.ToString(), out var guid))
            {
                return guid;
            }
          }
        }
      }
    }
  }
}
```

As proven by benchmarks provided with the PR, this change might be a game changer for big messages. After the change, the memory overhead for finding this property is constant. The same can be applied to the execution time. Before this change the allocation was strongly correlated with the size of the message not to mention the bigger time needed for bigger messages.

### Summary

These are three PRs that were provided by me and merged by the awesome team behind Azure Functions. As with every case of performance related investigations, measurement was the key, even in cases where the intuitive educated guess was clear. I hope that Azure Functions users will benefit from these changes and will be able to run more for less.
