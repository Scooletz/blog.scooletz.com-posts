---
layout: post
title: "Git plumbing with PowerShell"
date: 2015-12-09 11:00
author: scooletz
permalink: /2015/12/09/git-plumbing-with-powershell/
nocomments: true
categories: ["Git", "PowerShell"]
tags: ["development", "git", "git flow", "powershell"]
imported: true
---

In [the recent post](http://blog.scooletz.com/2015/12/04/protect-development-at-all-cost/) I've shown the need of securing the development branch in GitFlow. The same should be applied to all release branches and hotfixes as well. To provide a proper protection we need tooling and ability to gather some information from the git repository. Fortunately, Git structure is extremely easy and as shown in the very best [Git book chapter](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain), consists of a few atoms, elements, which because of their simplicity let us to create powerfull approaches like GitFlow. For sake of reference, this post won't cover whole plumbing but only the needed parts.

If you have any Git repo on disk, that's the time to open the hidden folder named .git. Don't be shy, just list its content. You'll see a structure which beside other elements contains following:

1. /refs

    1.  /heads

    1.  your branches are here!

    2.  /remotes

    1.  /origin

    1.  origin branches go here!

    3.  /tags
1. HEAD

Let's start with the HEAD (although Joker says no). Open your PowerShell and use simple *cat* or *Get-Content* its content. You'll see something similar to these:

*ref: refs/heads/feature/blob-management*

So that's how Git holds the reference to the current branch you're working on! With the following script you can easily obtain its full name.

*function Git-GetHeadFullReference(){* *return ((gc .git\HEAD).Replace("ref: ", "").Replace('/', '\'))* *}*

That was easy, wasn't it?

Now let's try to get all the names of features branches, having in mind the fact, that they are prefixed with the "feature/" string. We already know that all the feature branches are stored as files in *.git/refs/heads/feature/* For sake of security that someone added a subdirectory for any feature, we need to go recursively down, but skipping directories (if there's a directory, the real branch is below).

If you take a look and open any file representing branch, you'll find 40 characters. You probably know what is it. It's the identifier of the commit the branch is pointing at! Isn't it simple and beautiful? A branch is just a file with the identifier of the commit. Now you know why branching in Git is so cheap! Every branch is a single small file! Going back to our script let's enhance it with ability to return not only names of the branches, but identifiers of the commits they're pointing to as well.

*function Git-GetAllFeatureBranches(){* *return gci .\.git\refs\heads\feature -Recurse |* *where { ! $_.PSIsContainer } |* *Select-Object -Property @{n='name'; e={$_.Name}}, @{n='sha1'; e={(gc $_.FullName)}}* *}*

With a few lines of PowerShell we were able to obtain local branches with the commit identifiers. Admit that simplicity of this is so powerful!

Having this tooling, we'll be able to build up a nice fence around development, to keep it always green. Stay tuned!
