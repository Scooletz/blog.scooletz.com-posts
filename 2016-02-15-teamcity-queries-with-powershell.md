---
layout: post
title: "TeamCity queries with powershell (for sake of healthy develop)"
date: 2016-02-15 11:00
author: scooletz
permalink: /2016/02/15/teamcity-queries-with-powershell/
nocomments: true
categories: ["Git"]
tags: ["git", "TeamCity"]
imported: true
---

In the last entry I've shown how to create a PowerShell based git hook, passing following parameters to the hook for sake of validating it. The parameters were:

1. <span class="pl-s"><span class="pl-smi">*$local_sha* - the signature of the local commit</span> </span>

1. <span class="pl-s"><span class="pl-smi">*$remote_ref* - the remote branch name

</span></span>

1. <span class="pl-s"><span class="pl-smi">*$remote_sha* - the signature of the remote commit

</span></span>

This is more than enough to finalize our quest and test if the commit that is about to be pushed on develop was previously built in any other branch (in this case, a feature branch). To enable that we need a plan of attack. The check will consists of:

1. ensuring that *$remote_ref* name is equal to 'develop'

1. querying TeamCity server for the status of <span class="pl-s"><span class="pl-smi">*$local_sha*</span></span> in any of the feature branches

The first point is a one liner. The second point is a bit more interesting. To do not write the whole integration with TeamCity in PowerShell, we can use [TeamCitySharp ](https://github.com/stack72/TeamCitySharp)library, which integration with PS is so well described in [here.](https://blogs.endjin.com/2012/03/teamcitypowershell/) The easiest way to query for the specific commit build I've found so far, is to have a separate build for the features which enables to issue the following query against API (using [buildLocator](https://confluence.jetbrains.com/display/TCDL/REST+API#RESTAPI-BuildLocator))

> http://teamcity:8111/httpAuth/app/rest/builds/?locator=<buildLocator>

This list doesn't provide all the needed information. So for each build in there, we need to query for more details. Once we got the build details, you can see that they include the SHA1 of the commit being build, which is exactly what we wanted.

This post ends this series of protecting the development branch. I hope it will help you to adjust and improve rules in your environment to decrease the number of failed builds on development and increase overall quality of your solution.
