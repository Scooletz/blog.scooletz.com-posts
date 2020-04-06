---
layout: post
title: "Git hooks with PowerShell"
date: 2015-12-11 11:00
author: scooletz
permalink: /2015/12/11/git-hooks-with-powershell/
categories: ["Git", "PowerShell"]
tags: ["git", "git flow", "hooks", "powershell"]
imported: true
---

It's time to provide some client-side hooks for our Git repository! Git hooks are stored in the .git directory under following path:

*.git/hooks*

As you can see there's plenty of them, ending with *.sample.* When we remove this extension, the hook will kick in, handling events according to its name. There's an issue with Git hooks. They're stored under .git folder which is not versioned on its own, so you need to apply them locally, right? Not exactly. You can use following method, to have hooks version in the same repository as the code. Just copy the hooks to the repository level (above .git folder, to the root of the repo) and place a link to the folder above in the .git folder. Once it's done, git will use versioned hooks, from your repository. This enables you to treat hooks as a part of your repo and work with them as with any feature of your product.

Having established the hooks foundation, we can take a look at the *pre-push* hook. Just to cut this *bash* let me call PowerShell script with *exec*. This results in passing the whole execution directly to PowerShell, additionally handling *exiting* from ps script properly (it's passed to bash).

https://gist.github.com/3c34adf4f378435ca112

The important thing to mention is that this script will be called for **every branch you push**. That's why it's important to properly pass parameters.

If the PowerShell script returns non-zero code, it's treated as an error and push fails. This means, that we can validate the push against any source and fail it on purpose when some conditions are not met.

Summing up, you've seen so far a bit of Git plumbing and a way how to turn pre-push hook into script calling your Prepare-Push function from a script. As you probably remember, that's the place when we're going to validate the push quality against some service. In our case it's going to be *TeamCity API.*
