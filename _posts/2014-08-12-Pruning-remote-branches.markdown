---
layout: post
title:  "Pruning Remote Branches"
date:   2014-08-12
categories: git
---

Recently I started to use git in a more professional setting.  I have been using it for several years on my own but things change once you start to use it in a team setting.  Once you begin using it in the team setting branch management  starts to become an issue. 

Yesterday I created a branch that was going to merged directly into the master branch and deployed once I was done developing it.  It was a small feature so I was complete with development in a few hours.  A teammate was going to review the branch so I pushed the branch to the remote repository for him to pull down. This was when we ran into our first problem.  

## Seeing New Remote Branches
My teammate  ran ```git remote -a``` and was not listed with the remote branch I had just created.  Where to did it go?

Git tracks the remote branches locally and so you have to refresh your tracking list (this is important to remember for later).  To be able to see the new remote branches he needed to run ```git fetch```.  Fetch updates the your locally tracked remote branches.  Think of it like a local cache of remote branches.  After awhile you need to refresh your latest list.

##  Removing Old Branches
Now that he was able to see the branch he reviewed it and merged it into the trunk.  He went ahead and deleted his local branch (```git branch -d branchname```) and the remote branch (```git branch :branchname```) for the feature and I went back to my desk.  

When I got there I pulled down the new trunk changes.  I expected this would refresh my list of repository's and I would not see the feature branch my teammate just removed.  I was wrong.  When I ran ```git remote -a```  I found the remote feature branch was still listed.  

I was a bit confused, but then remember that git tracks branches locally.  After a quick google search I found my answer.  To refresh your local remote branch list you need to run the command ```git fetch --prune```.  

## Automating
Don't want to remember to do that every time? While doing the research into remote branches I found that as of version 1.8.5 you can configure git to do this automatically for you. Here is the configuration:

```
git config fetch.prune true
```

You can learn more about it from Alberto Grespan's blog post [Always prune remote-tracking branches](http://albertogrespan.com/blog/always-prune-remote-tracking-branches/)

I hope this short story helps you understand how you might end up with to many remote branches in your local repository and how to clean them up.

