---
layout: post
title: "Git repository as a graph"
date: 2014-04-26 11:00
author: scooletz
permalink: /2014/04/26/git-repository-as-a-graph/
nocomments: true
categories: ["Git"]
tags: ["git", "graph", "repository"]
imported: true
---

One of the greatest misunderstanding of git is trying to map it 1-1 to SVN. It's easy to follow this fallacy when one starts migrating from SVN and all he/she fears the most is loosing the precious branches and all the rituals connected with them. Let's state from the beginning:

### Don't be stupid. Learn Git. Stop trying mapping all SVN stuff to the new environment.

A simpler abstraction of git repo can be started from a graph of commits. Consider each git commit object (see [here](http://git-scm.com/book/en/Git-Internals-Git-Objects) if you have no idea what kinds of object are present in git) connected with its parents by an oriented graph edge. A git repository can be considered then as a graph. Under normal circumstances that graph is a [directed acyclic graph](http://en.wikipedia.org/wiki/Directed_acyclic_graph):

* directed - a parent of a commit is a head of an edge and the commit is its tail
* acyclic - as there are no returns to the earlier states, no cycles in the history

What is a branch then? It's a named pointer to a given commit. It's not a folder or any existing chain of commits. It's a pointer sliding in time to the children commits. Do not attach more meaning to it.

What is a [fast forward merge](http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging) then? It's a merge possible only if one of two commits being merged can be moved along the graph path to the second one.

One can try to preserve a linear history over one branch but as a directed acyclic graph defines only partial order, that may be impossible. Remember tagging important moments in history of a repository. Once the branch pointer was moved away, there may be no turning back - merge commit doesn't have distinguishable parents.

To sum up. Don't use improper abstractions over git. A repository should be considered as a directed acyclic graph. Use tags as breadcrumbs to point important events in history.
And use git for Linus sake!
