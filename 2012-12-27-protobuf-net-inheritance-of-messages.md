---
layout: post
title: "Protobuf-net: inheritance of messages"
date: 2012-12-27 16:45
author: scooletz
permalink: /2012/12/27/protobuf-net-inheritance-of-messages/
nocomments: true
categories: ["Protobuf-net"]
tags: ["inheritance", "serialization"]
imported: true
---

The [last post](http://blog.scooletz.com/2012/12/17/protobuf-net-message-versioning/) was an introduction to a simple project called Protopedia, located [here](https://github.com/Scooletz/Protopedia). The project is destined to bring in a simple manner, probably one test per case, solutions for complex scenarios like versioning, derivation of messages, etc. As the versioning was described by the previous entry, it's right time to deal with derivation.

### Inheritance

It's well known fact that one should favor composition over inheritance. Dealing with derivation trees with plenty of nodes can bring any programmer to his/her knees. How about messaging? Does this rule apply also in this area? It's common for messages to provide a common denominator, containing fields common for all messages (headers, correlation identifiers and so on), especially if they're meant to be sent/saved as a stream of messages of the base type (example: Event Sourcing with events of a given aggregate). Using a set of messages with a distilled root greatly simplifies concerns mentioned earlier. Consider the following scenario, serialization of a collection of A messages (or its derivatives) being given the following structure:

![Message inheritance tree for example ](http://yuml.me/8c92fa40)

How would Protobuf-net serialize such collection? First, take a look at the [folder from Protopedia](https://github.com/Scooletz/Protopedia/tree/master/Protopedia/Derivation). You can notice, that all the classes: A, B, C, have been mapped with different types. It's worth to notice the ProtoInclude attributes with tag values of the types located one level deeper in the derivation tree. The second important thing is the values of the derived type tags, which do not collide with tags of the class fields. In the example, you can find a constant value of 10 used for sake of future versions of the root, the A class. As one can see in the test of the derivation, the child classed of the given class are serialized as fields with the tags equal to the tag passed in the ProtoInclude attribute. To see the fields composed in a way the Protobuf-net serializes inherited messages take a look into [following message contracts](https://github.com/Scooletz/Protopedia/blob/master/Protopedia/Derivation/MessagesWithCompositionInsteadOfInheritance.cs). There's no magic and the whole idea is rather straightforward: serialize derivatives as fields, turning the inheritance into the composition. This working proposal of Protobuf-net will be sufficient and effective in all of your efforts of serialization of inheritance. Nice serializing!
