---
layout: post
title: "Ping pong Bruce Lee test"
date: 2016-04-08 10:00
author: scooletz
permalink: /2016/04/08/ping-pong-bruce-lee-test/
categories: ["RampUp"]
tags: ["RampUpNet"]
imported: true
---

![](https://images.duckduckgo.com/iu/?u=http%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M288e84fa6a398a3beb9bd259126fde65H0%26pid%3D15.1&f=1)There is a famous Bruce Lee clip showing him as a very good ping pong player using unusual tooling to get the job done. It thought that this ping pong match would be a great story for writing a test for my [RampUp](https://github.com/Scooletz/RampUp) library, especially when I provided the first, most likely not final, version of the actor system.

To have more fun I split Bruce Lee into Bruce &amp; Lee. Each part of Bruce Lee either pings or pongs.

[code language="csharp"]

public class Bruce : IHandle<Ping>
{
    public IBus Bus;

    public void Handle(ref Envelope envelope, ref Ping msg)
    {
        var p = new Pong();
        Bus.Publish(ref p);
    }
}

public class Lee : IHandle<Pong>
{
    public IBus Bus;

    public void Handle(ref Envelope envelope, ref Pong msg)
    {
        var p = new Ping();
        Bus.Publish(ref p);
    }
}
```

The ping/pong messages are only markups:

[code language="csharp"]

public struct Pong : IMessage {}

public struct Ping : IMessage {}

```

And the final execution of this setup can be summarized in:

[code language="csharp"]

public class Program
{
    public static void Main()
    {
        var system = new ActorSystem();
        IBus bus = null;

        system.Add(new Bruce(), ctx => { bus = ctx.Actor.Bus = ctx.Bus; });
        system.Add(new Lee(), ctx => { ctx.Actor.Bus = ctx.Bus; });

        system.Start();

        var p = new Pong();
        bus.Publish(ref p); // pong as Bruce
        // ... later
        system.Stop();
    }
}
```

I hope you like the example. I'm aware that *ActorSystem* API isn't the best possible API ever, but even in this shape enables me to push RampUp forward.
