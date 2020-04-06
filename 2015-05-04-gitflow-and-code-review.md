---
layout: post
title: "GitFlow and code reviews"
date: 2015-05-04 10:00
author: scooletz
permalink: /2015/05/04/gitflow-and-code-review/
categories: ["Code review", "Git"]
tags: ["code review", "git", "git flow"]
imported: true
---

[![Presentation1](/img/2015/05/presentation11.png)](/img/2015/05/presentation11.png)Many companies that uses GIT as its code repository use [git-flow](http://nvie.com/posts/a-successful-git-branching-model/) as the branching model for its projects. The question, that one can come up with is where and when the code review should be taken? Should it be before:

*git flow feature finish MYFEATURE*

If yes, then the reviewer looks through some version of code, but still, the person responsible for closing the feature creates a a new merge commit after all which can change a lot. On the other hand, if the reviewer creates the merge commit, he/she may not know all the aspects needed for a successful merge. There are a few pages trying to answer this, but still I haven't found anything satisfying. Please read further for my proposal of clean and proper code review process with git-flow.

The problem with git-flow is the fact that finishing a feature is an atomic action. In one action the author does plenty of things

1. Author: pushes the commit to the remote
1. Reviewer: reviews the code
1. A: creates a merge commit (**new commit created!**)
1. A: **moves the develop** cursor to the new commit

1. A: deletes the feature branch
1. A: pushes to the remote

As always, splitting one action into multiple and then proper grouping might help. Consider the following flow including the author and the reviewer.

1. Author: creates a merge commit
1. A: **moves the feature/a** to the merge commit

1. A: pushes the merge commit to the remote
1. Reviewer: reviews the merge commit
1. R: if the review is successful, **rebase the develop with fast-forward** to the merge commit (some automation can be introduced).

According to this flow, the reviewer always reviews the commit which will land in the develop. Additionally, this makes the author of the feature branch aware of merge problems before he/she pushes out his/her work to the review. Effectively, a feature can be completed and merged and simply wait for acceptance which then, is a simple "go/no go" without considering merge conflicts later on.

This isn't a git-flow anymore, but still, it is a solid flow to lower the context switching of the author. Ones he/she finishes, it's a real end, not an entry for incoming merge.
