---
layout: post
title:  "A Short Story of Pruning Remote Branches"
date:   2014-08-14
categories: git
---

Recently I started to use git in a more professional setting.  I have been using it for several years on my own but things change once you start to use it in a team setting.  Once in a team setting, branch management becomes something you have to pay attention too. 

Yesterday I created a branch that was going to merged back into the master branch and deploy once I was done developing it.  A teammate was going to review the branch and deploy, so when I was done with development I pushed the branch to the remote repository for him to pull down. This was when we ran into our first problem.  

## Seeing New Remote Branches
My teammate  ran 

```git remote -a``` 

This lists remote branches but the remote branch I had just created.  Where to did it go?

*Git tracks the remote branches locally and so you have to refresh your tracking list* (this is important to remember for later).  To be able to see the new remote branches he needed to run:

 ```git fetch```

Fetch updates the your locally-tracked remote branches.  Think of it like a local cache of remote branches.  After awhile you need to refresh your latest list.

##  Removing Old Branches
Now that he was able to see the branch, he reviewed it and merged it into the trunk. Using good branch management, he deleted his local branch and the remote branch for the feature:

```
// delete local
git branch -d branchname 
// delete remote
git branch :branchname
```

When I got back to my desk I pulled down the new trunk changes.  I expected this would refresh my list of repository's and I would not see the feature branch my teammate just removed.  I was wrong.  When I ran ```git remote -a```  I found the remote feature branch was still listed.  

I was a bit confused, but then remember that git tracks branches locally.  After a quick google search I found my answer.  To refresh your local remote branch list you need to run the command: 

```git fetch --prune```

## Automating
Don't want to remember to do that every time? While doing the research into remote branches I found that as of git version 1.8.5 you can configure git to do this automatically for you. Here is the configuration:

```
git config fetch.prune true
```

You can learn more about it from Alberto Grespan's blog post [Always prune remote-tracking branches](http://albertogrespan.com/blog/always-prune-remote-tracking-branches/)

I hope this short story helps you understand how you might end up with to many remote branches in your local repository and how to clean them up.

