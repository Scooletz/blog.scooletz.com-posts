---
layout: post
title: "Capability-based security"
date: 2020-06-08 11:00
author: scooletz
permalink: /2020/06/08/capability-based-security
image: /img/2020/05/capability-based-security.png
categories: ["security", "capabilities", "cloud"]
tags: ["security", "capabilities", "cloud"]
whitebackgroundimage: true
---

This post is meant to deliver you a dense and meaningful primer on the capability-based security. I'm far from being expert in this topic. What I did I spent time trying to read a lot, understand it and then map it to a form that should be understandable and interesting to read. I hope you'll enjoy it and learn a lot. I follow a do and don't mantra, summarizing each section, to help you build a valid mental model of what capabilities are and what they are not.

### The fallacy of the matrix

One of the common approaches for addressing security are ACLs, which stands for Access Control Lists. They provide a mapping between a list of identities and the operations the process running unders a specific identity can perform. To make it clearer, one could draw a table to show how Alice, Bob and Malory (we're in the security land, we need Malory to have some fun) can interact with some files.

|   | file.txt  |  file2.txt | secrets.txt  |
|---|---|---|---|
| Alice   | RW  | R  | R  |
| Bob  | -  |  R  | R  |
| Malory  | -  | -  | RW |

As we can see, the columns in here could be a representation for ACLs. Reading them, we have a clear list of identities with their rights. This is a kind of problem, as it can be seen that every object in the system needs to know about identities and access rights. Let's assume that this is not the biggest error to assume it.

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

What if the capability could be passed from Alice when requesting something from Bob? What if Alice passed her capability to read to file.txt when calling Bob for something? This kind of dynamic model is not addressable by a simple row or a column. It shows that the matrix representation, which is a static one, can describe a snapshot of reality, but cannot describe a complex behavior which might emerge from passing on a capability.

So far we observed that:

- a capability can be possessed
- a capability can be passed it elsewhere, enabling the callee to use the passed capability.

### Who are you anyway

### The data attack

Capabilities are often referred to as:

1. unforgeable tokens
1. unforgeable keys

### Capabilities as objects

### Systematic separation

### Capabilities in the cloud

### Bibliography

This article was created on the basis of a short-term study of the following sources. I did my best to aggregate the information properly and if you find any mistake, you can probably attribute it to me.

1. [Levy book](http://www.cs.washington.edu/homes/levy/capabook)
1. [Deconstructing process isolation](https://dl.acm.org/doi/10.1145/1178597.1178599)
1. [Robust Composition:Towards a Unified Approach to Access Control and Concurrency Control](http://www.erights.org/talks/thesis/markm-thesis.pdf)
1. [Capability Myths Demolished](https://srl.cs.jhu.edu/pubs/SRL2003-02.pdf)
1. [Protection and Access Control in Operating Systems](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/11/09-ProtectACinOS.pdf)
1. [Membranes](https://tvcutsem.github.io/membranes)
1. [A taste of Capsicum: practical capabilities for UNIX](https://dl.acm.org/doi/10.1145/2093548.2093572)
1. [Introducing Capsicum: practical capabilities for UNIX](https://www.cl.cam.ac.uk/research/security/capsicum/papers/2010usenix-login-capsicum.pdf)