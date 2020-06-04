---
layout: post
title: "Capability-based security"
date: 2020-06-08 11:00
author: scooletz
permalink: /2020/06/08/capability-based-security
image: /img/2020/06/capability-based-security.png
categories: ["security", "capabilities", "cloud"]
tags: ["security", "capabilities", "cloud"]
whitebackgroundimage: true
---

This post is meant to deliver you a dense and meaningful primer on the capability-based security. I'm far from being expert in this topic. What I did was spending some time trying to read a lot, understand it and then map it to a form that should be understandable and interesting to read. I hope you'll enjoy it and learn a lot. I follow a do and don't mantra, summarizing each section, to help you build a valid mental model of what capabilities are and what they are not.

Huge kudos go to [Marcin Hoppe](https://twitter.com/marcin_hoppe), for sharing his security wisdom with me and showing different angles of capability-based security.

### The fallacy of the matrix

One of the common approaches for addressing security are ACLs, which stands for Access Control Lists. They provide a mapping between a list of identities and the operations the process running under a specific identity can perform on a resource ([wiki article](https://en.wikipedia.org/wiki/Access-control_list) describing a single ACL as _a list of permissions attached to an object_). To make it clearer, one could draw a table to show how Alice, Bob and Malory (we're in the security land, we need Malory to have some fun) can interact with some files:

|   | file.txt  |  file2.txt | secrets.txt  |
|---|---|---|---|
| Alice   | RW  | R  | R  |
| Bob  | -  |  R  | R  |
| Malory  | -  | -  | RW |

As we can see, the columns in here could be a representation for ACLs. Reading them, we have a clear list of identities with their rights. This could be perceived in a way, that every object in the system needs to know about identities and access rights. Let's assume that this is not the biggest thing and let' move further.

Let's take a look at the rows though. How can they be read? It looks like:

1. Alice has a capability:
    1. to Read and Write to file.txt
    1. to Read from file2.txt
    1. to Read from secrets.txt
1. Bob  has a capability:
    1. to Read from file2.txt
    1. to Read from secrets.txt
1. Malory has a capability:
    1. to Read and Write from secrets.txt

This looks like exactly the same information shown in two different ways, but as [Capability Myths Demolished](https://srl.cs.jhu.edu/pubs/SRL2003-02.pdf) shows

> However, such a line of reasoning ignores a crucial factor: the difference between static models and dynamic systems.

What if the capability could be passed from Alice when requesting something from Bob? What if Alice passed her capability to read from file.txt when calling Bob for something? This kind of dynamic model is not addressable by a simple row or a column. It shows that the matrix representation, which is a static one, can describe a snapshot of reality, but cannot describe a complex behavior which might emerge from passing on a capability.

So far we observed that:

- a capability can be possessed
- a capability can be passed to another entity, enabling the callee to use the passed capability

### Who are you anyway

One of the major problems in security is so called _ambient authority_, which is an implicit assumption of running a process under an account that started the process. Of course, various operating systems enable you to limit the privileges or run processes under a different account, but the default is the same: the process inherits privileges of the account.

Capabilities are different in a way, that in majority of systems they are passed to the callee, sometimes being limited, as it's always possible to subtract from a set of capabilities, pass just some of them. In [Levy book](http://www.cs.washington.edu/homes/levy/capabook) you can find a lot of examples where various systems took different path for calling so called _system procedures_, that usually require higher level of privileges. This can be addressed by the procedure itself keeping a list of its own capabilities and acting upon the set combined from the caller capabilities and their own.

The fact that there's no privilege level (no rings), just capabilities and their sets enabled [Singularity and then Midori](https://dl.acm.org/doi/10.1145/1178597.1178599) to totally remove rings and virtual memory mapping. All the processes, that run programs verified by a specific compiler, could be possible to run in software isolation, just passing messages and enjoying full isolation at the same time.

So far we observed that:

- a procedure can have its own list of capabilities
- capabilities remove the _ambient authority_ by replacing it with capability-passing
- capabilities can simplify or even remove the distinction between privileged and unprivileged calls.

### Data vs objects

Capabilities are often referred to as **unforgeable tokens** or **unforgeable keys**. This is a perception that if followed literally (token or a key can be stored) can bring a data attack. Assume the following scenario:

1. **Alice** has a capability:
    1. to Read and Write to notes.txt
    1. to Read and Write to all-my-secrets.txt
1. **Malory**  has a capability:
    1. to Read from notes.txt

Now assume that

1. capability can be written down as you would write an encryption key or API key
1. Alice writes down its capability for reading and writing in all-my-secrets.txt to notes.txt
1. Malory reads notes.txt

💥 We've just violated security properties of the system by allowing to store capabilities. Therefore treating them as tokens or keys is somewhat flawed. They should be perceived as objects that you can act upon and use their... capabilities. This the very same reason why all the systems described so well in [Levy book](http://www.cs.washington.edu/homes/levy/capabook) take so much care for disallowing to pass or store "the address" of a capability. This is why thinking about them as objects, like in a regular object-oriented programming is so much better.

So far we observed that:

- a capability should be treated as an object
- a capability cannot be stored

### Indexing and tagging

If the capabilities cannot be stored, how a process can address a capability or store information about a file that it wants to write to? In systems that do not deal with persistence explicitly and use [orthogonal persistence](https://en.wikipedia.org/wiki/Persistence_(computer_science)#Orthogonal_or_transparent_persistence), it's not a problem. You don't write or read data anyway and they are stored for you in the background. What about systems using explicit operations to store data?

In systems where the storage/retrieval of data is explicit, the indexing or tagging can be used. If you think about capabilities being stored in an array, you could use an index in the array, assuming that the location of a capability is not changed. Once you write it to a file and read again, you'll be able to read it again from the list. Consider the following example:

Process A can:

1. read and write to _file 1.txt_
1. write to _2.txt_

Both these capabilities are stored in a list that can be addressed by an index. If process A wants to store information about the file that it want to write to, assume it's 2.txt, it can store the index equal 1 (numbering from 0) to "remember the file" that is the output of the operation. The stored index is meaningless for any external process and has a meaning only in the context of process A which can map it to the capability. This does not leak the capability itself as mentioned in the attack above.

So far we observed that:

- a capability can be given an index to have it persisted instead of the capability itself

### Capsicum

A really interesting approach for introducing capabilities in a non-breaking way to an existing operating system is [Capsicum](https://www.cl.cam.ac.uk/research/security/capsicum). The project can be used on FreeBSD, Linux and DragonFlyBSD.

The way it enables capabilities is a special call that to a system procedure called `cap_enter`. It sets a process credential flag that is inherited by all descendent processes and cannot be cleared. If a process is in capability mode, it is denied access to global namespaces such as the file system, PIDs and more.

> Sidenote: One could argue that _the process in capability mode is essentially a sandbox that is only capable of accessing resources it has a capability for_. Actually, I wanted to write it like this, but in the Capsicum papers (not peppers 😂) authors used the exclusive phrasing. As I'm not that familiar with Linux Kernel, I wanted to stick to exclusion of resources rather than claiming that it's capabilities only. Being given that verifying it would take much more, I'll stick to the phrasing as it is.

The same works for system procedures calls. Some of them are totally restricted while others accept different parameters. What kind of parameters are they?

Capsicum introduces a level of indirection wrapping file descriptors into capabilities. This means that for the same file, that was opened with read and write permissions, different capabilities objects can be created and passed down, to other procedures/child processes. The important fact is that like other file descriptors, capabilities may be inherited across `fork` and `exec`, as well as passed via UNIX domain sockets. This makes them an in-place replacement for the regular file descriptors.

The important part of the implementation is that most constraints are applied in the implementation of kernel services. Imagine that you can implement a single constraint in one place, which is responsible for processing all calls from user processes. This is done by providing a conversion from capability back to struct file reference that has additional rights added to represent a capability. With this, it can be checked in place.

### Membranes

Another approach, somewhat similar to capabilities are JavaScript membranes. Imagine that you create a bubble around an object representing a directory that is capable only of reading from this directory. What if you requested a file from this bubble? How should it behave? You'd probably assume that it should be wrapped in a bubble like the directory (imagine a moment when a cell divides) and it would allow only a specific set of actions.

In programming terms, you could think of it as an interface that has methods returning other interfaces. Never a concrete underlying construct.

### The genesis of this entry, a.k.a. towards the distributed

The whole journey in understanding capabilities was based on posts about [Midori system](http://joeduffyblog.com/2015/11/10/objects-as-secure-capabilities/) by Joe Duffy. An amazing post made me think about various things. 

Now comes the sketchy part of this post.

What if it is the way for the serverless. Imagine having functions that are much less boilerplate and that can accept meaningful parameters like `IResponder`. What if this responder could be passed to another functions? What if there was a was for the original caller to revoke the responder. If you take a look at Azure ecosystem now, you can do everything or nothing. There's no way to pass an object (not a token) down that can be acted upon. As always this could be solved by another layer of abstraction, a creation of a gateway/keeper that would keep capabilities and protect them from treating them as data.

I'm not sure if this would be the best idea, as Joe states in his blog post about an operating system:

> We never did get much real-world exposure on this model. The user-facing aspects were under-explored compared to the architectural ones, like policy management. For example, I doubt we’d want to ask my mom if she wants to let the program use a Clock. Most likely we’d want some capabilities to be granted automatically (like the Clock), and others to be grouped, through composition, into related ones. Capabilities-as-objects thankfully gives us a plethora of known design patterns for doing this. We did have a few honey pots, and none ever got hacked (well, at least, we didn’t know if we did), but I cannot attest for sure about the quantifiable security of the resulting system.

Not to mention planet scale distributed serverless apps.

### Summary

As a summary, let's got through the capability-based points again:

- a capability can be possessed
- a capability can be passed to another entity, enabling the callee to use the passed capability
- a capability should be treated as an object
- a capability cannot be stored
- a capability can be given an index to have it persisted instead of the capability itself
- a procedure can have its own list of capabilities
- capabilities remove the _ambient authority_ by replacing it with capability-passing
- capabilities can simplify or even remove the distinction between privileged and unprivileged calls.

I hope you enjoyed this ride and that it showed you various aspects of capability-based security.

### Bibliography

This article was created on the basis of a study of the following sources. I did my best to aggregate the information properly and if you find any mistake, you can probably attribute it to me.

1. [Levy book](http://www.cs.washington.edu/homes/levy/capabook)
1. [Deconstructing process isolation](https://dl.acm.org/doi/10.1145/1178597.1178599)
1. [Robust Composition:Towards a Unified Approach to Access Control and Concurrency Control](http://www.erights.org/talks/thesis/markm-thesis.pdf)
1. [Capability Myths Demolished](https://srl.cs.jhu.edu/pubs/SRL2003-02.pdf)
1. [Protection and Access Control in Operating Systems](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/11/09-ProtectACinOS.pdf)
1. [Membranes](https://tvcutsem.github.io/membranes)
1. [A taste of Capsicum: practical capabilities for UNIX](https://dl.acm.org/doi/10.1145/2093548.2093572)
1. [Introducing Capsicum: practical capabilities for UNIX](https://www.cl.cam.ac.uk/research/security/capsicum/papers/2010usenix-login-capsicum.pdf)
1. [Robust Composition:Towards a Unified Approach to Access Control and Concurrency Control](http://www.erights.org/talks/thesis/markm-thesis.pdf) I didn't read this one now, but it might be read later.
