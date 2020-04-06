---
layout: post
title: "Merge request policy"
date: 2014-04-23 11:00
author: scooletz
permalink: /2014/04/23/merge-request-policy/
nocomments: true
categories: ["Git"]
tags: ["flow", "git", "github", "gitlab", "merge request", "pull request"]
imported: true
---

If you're in a company with multiple teams in IT department, you could've been considering using [GitHub Enterprise](https://enterprise.github.com/) or its free replacement, [GitLab](https://www.gitlab.com/). Beside providing Git hosting experience both can support you with one aspect previously unknown to your organization: pull/merge requests.

A pull request in GitHub, a merge request in GitLab provides the same functionality. They let other service users to issue demands of changes in a way that makes it easy to apply onto the original repository. The advantages of using this kind of change management over sending diffs with emails or other ways of applying fixes in other codebases are:

1. Traceability - a request has its URL, is linkable and is public
1. Permission granularity - everyone can be given a permission to read and fork repo but not to write. This lets the owners to stay owners and deal with issued requests rather than a broken codebase because of mistakes made by others (for particular flows, read [here](http://git-scm.com/book/en/Distributed-Git-Distributed-Workflows))
1. True ownership - an owner is released from digging the dirt and moved to the acceptor state
1. No more emails - email requests are no longer sent. The basic way of asking for a given change is... doing the change
1. Learning over abusing - the requester is given an opportunity to leave with other codebase. It can cost a bit more, that's for sure, but it spreads practices and knowledge. The most important thing for owner is to help to create pull requests, not to apply them onto repositories.

This kind of change can be painful for lazy people, which want to delegate, or rather, push away all their work with no insight about the requested changes. This can be a big learning opportunity as well. Adopting OSS rules, like merge-request-policy in your day-to-day job can increase your awareness and make you a happier developer.

It's time to issue some pull requests!
