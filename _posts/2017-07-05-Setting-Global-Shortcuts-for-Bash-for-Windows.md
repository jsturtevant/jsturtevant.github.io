---
layout: post
title: Setting Global Shortcuts for Bash for Windows
date: "2017-07-05"
categories:
  - windows
---

I have started using [Bash for Windows](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) as part of my workflow.  For a long time, I have been using the standard command prompt with [Git Bash tools](https://git-for-windows.github.io/) hooked up to my path (using [chocolately to install](http://www.jamessturtevant.com/posts/Chocolatey-And-Boxstarter/) the tools in one command `choco install git -params '"/GitAndUnixToolsOnPath"'`). This was nice because I could use the shortcut `Windows Key + X` then `C` to open a command prompt and get access to Linux like tools such as `ls`, `git` and `grep`.  

Now that I am using Bash for Windows I wanted to have a shortcut set up as well.  I found it was really easy to set up using the `Shortcut key` setting on the shortcut properties window.  This process could be used for any application that has a shortcut setup.  The basic steps are:

1. Find the shortcut by `Windows Key` the type `bash`
2. Right click on the shortcut and select `open file location`

![find shortcut for Bash for Windows]({{ site.url }}/assets/bash-for-windows-find-shortcut.png)

3. Right click on the shortcut in the folder and select `Properties`
4. Select `Shortcut key` and type your keyboard shortcut

![setup global shortcuts for Bash for Windows]({{ site.url }}/assets/bash-for-windows-shortcut.png)