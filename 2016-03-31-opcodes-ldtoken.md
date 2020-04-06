---
layout: post
title: "OpCodes.Ldtoken"
date: 2016-03-31 09:00
author: scooletz
permalink: /2016/03/31/opcodes-ldtoken/
nocomments: true
categories: ["C#"]
tags: ["MSIL", "RampUpNet"]
imported: true
---

You probably used *typeof* operator a few times. It's quite funny, that an operator like this actually has no MSIL counterpart. Using it emits TWO [OpCodes](https://msdn.microsoft.com/en-us/library/system.reflection.emit.opcodes%28v=vs.110%29.aspx).

The first emitted opcode is OpCodes.Ldtoken with the type. It consumes the token of the type, pushing the [RuntimeTypeHandle](https://msdn.microsoft.com/en-us/library/system.runtimetypehandle%28v=vs.110%29.aspx) structure onto the stack as the result of its operation. The second emitted code is a call to the [Type.GetTypeFromHandle(RuntimeTypeHandle)](https://msdn.microsoft.com/en-us/library/system.type.gettypefromhandle%28v=vs.110%29.aspx) which consumes the structure pushed by the previous code, returning the runtime type. The interesting thing is that you can't use just *OpCodes.Ldtoken* from C#. You need to load the runtime type first and then you can access the handle by a property. You can emit IL with just OpCodes.Ldtoken though to remove overhead of calling a method and use the structure as a key for a lookup. It will be a bit faster for sure.

You can see the example of emitting this in RampUp code of the [message writer](https://github.com/Scooletz/RampUp/blob/master/src/RampUp/Actors/Impl/BaseMessageWriter.cs#L90).
