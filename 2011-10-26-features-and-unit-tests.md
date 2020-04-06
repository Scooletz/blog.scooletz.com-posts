---
layout: post
title: "Features and unit tests"
date: 2011-10-26 11:00
author: scooletz
permalink: /2011/10/26/features-and-unit-tests/
nocomments: true
categories: ["Design", "TDD"]
tags: ["design by feature", "unit tests"]
imported: true
---

Currently I'm involved in a greenfield project, which was happy enough to start as a event-sourced DDD. The paradigm fits well the domain, but there is some cost of zero-iteration which is being paid now. The cost is a small infrastructure which has to be provided to be wrap the domain. It's really small (maybe [not that small](https://github.com/gregoryyoung/m-r "Greg's m-r")), but has to be tested. To make it simple/complex I'd add that it consists of a few functionalities/modules, each providing and consuming some contracts. So what about testing?

Recently I re-read [a Kozmic's blog entry](http://devlicio.us/blogs/krzysztof_kozmic/archive/2011/02/28/unit-tests-are-overrated.aspx "Unit tests are overrated"), which covers a very interesting problem of unit tests vs 'real life scenarios' as well as ['Concepts and features'](http://ayende.com/blog/3895/application-structure-concepts-features "Concepts and features") written by Ayende. It made me think a lot about the new application and the way I tried to test it. I took the first module, which had a small test base written in a not-good-as-it-should-be manner, deleted the tests and started to implemented sth new.

I know that some people have opinion 'no container in your tests', but in tests of the infrastructure? Come on, this is all about matching all the pieces of the code you wrote! Nevertheless, having the WindsorInstaller of a specific part of the application I initialized the container, added *one* stub for the external dependencies and fully configured this part of the application. All the tests used the enpoints provided by the whole functionality and it worked like a charm. It seems that the number of test fixtures will be equal to the number of functionalities/modules the infrastructure provide. I find it a quick, simple and powerful solution for having it testes. What about the unit tests? If I have some infrastructure class longer than 100 lines and plenty of methods in there, I will go for unit tests, but for now, this paradigm works like a charm.
