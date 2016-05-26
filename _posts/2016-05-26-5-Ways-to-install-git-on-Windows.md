---
layout: post
title: 5 Ways to Install Git on Windows
date: "2016-05-26"
categories:
  - git
---

Git is a major part of a developers work flow these days no matter what platform you work on.  There are many different ways to install and use Git on Windows and in this post I will cover 5 different ways and talk about the pro's and con's of each.  At the end you should have a good idea of how to have a great experience with Git on Windows not matter what your scenario.  Be sure to let me know your favorite in the comments or if I missed one.   

There is a walk through video for each of the ways to install git on Windows at [5 Ways to install Git on Channel 9](https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows).

## The Five ways
Below are the 5 basic ways to install git on Windows. There are probably others but these are the ones that I tend to show developers.  I personally tend to end up installing git for command line usage *and* installing a GUI for more complex tasks.  There are a few tasks ([sqaushing commits](https://github.com/blog/2141-squash-your-commits)) that are easier on the command line and a few that are easier in a GUI (selecting single lines from multiple changes in a file for an atomic commit).  

1. [Git for Windows](#git-for-windows) 
2. [Using Chocolatey](#using-chocolatey)  - preferred way on Windows 10
3. [Using Cmder](#using-cmder)  - Great for older version of Windows
4. [Your Favorite Git GUI](#your-favorite-git-gui) - [Visual Studio](https://www.visualstudio.com/), [Sourcetree](https://www.sourcetreeapp.com/), [GitHub Desktop](https://desktop.github.com/), [GitKraken](https://www.gitkraken.com/), etc
5. [Bash on Windows](#bash-on-windows)  - In preview for Windows 10 (sign up for the [Windows Insider](https://insider.windows.com/) to try it out today)

## Git for Windows
[Git For Windows](https://git-for-windows.github.io/) is the foundation for running Git on Windows.  Many of the other options listed are using Git for Windows (previously msygit) under the hood.  This option will install the git client, the windows implementation of BASH tools and a few Git GUI tools.  If you want just the raw tools then this is the installer for you.  If you want some more advance tooling then look to some of the other options.  I generally prefer to install this via a different option.

You can install it from [Git for Windows](https://git-for-windows.github.io/).  The video walk-through is at [todo](https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows).

### Pro's

- basic tooling 
- shell integration (right click to open BASH Promp / GUI)

### Con's 

- GUI is old school
- default is to use a specific BASH prompt instead of standard command line
- have to use installer


## Using Chocolatey
This is my preferred way to install the Git for Windows on Windows 10.  It installs the same package before but in one line.  If you have not heard of [Chocolatey](https://chocolatey.org/) stop everything and [go learn a bit more](https://chocolatey.org/about).  Chocolatey can install the software with a single command; you don't have to use click-through installers any more! If you are coming from Linux think of it as the apt-get/yum for Windows. Chocolatey is very powerful and I use it in combination with [Boxstarter](http://www.boxstarter.org/) to [set up my dev machines](http://www.jamessturtevant.com/posts/Chocolatey-And-Boxstarter/).  If you are in charge of setting up machines for developers on windows it is definitely worth a look.

I love this option because it is fast to set up (one liner) and with [Windows 10's new command console](https://blogs.windows.com/buildingapps/2014/10/07/console-improvements-in-the-windows-10-technical-preview/) (it resizes and has copy/paste) I get the a great native experience.  Using keyboard shortcuts (```Windows key + X then C```), I can have a command prompt open quickly and start using git right away.  [Watch the video]((https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows)) to see another cool trick where you can open the command window in any open folder.

Let's see how you would install git using Chocolatey.  I assume you have [Chocolatey installed](https://chocolatey.org/install) already (it is a one liner in the command prompt). Then simply open Administrator Command Window and type:

```
choco install git -params '"/GitAndUnixToolsOnPath"'
```

This will install git, the BASH tools and add them to your path.  Check out the video to see a walk-through [TODO](https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows).

### Pro's

- fast setup
- use existing Windows 10 command prompt for native experience

### Con's 

- if done on older version of windows the command prompt experience is poor
- have to have Chocolately installed (though really easy to set up and over all awesome tool)

## Using Cmder
This was my go to option before Windows 10  and would be what I recommend for anyone not using Windows 10.  The command line experience in older versions of Windows is poor (no resize/copy paste) and [Cmder](http://cmder.net/) brings together a set of awesome tools to make a great command line experience.  I always choose the larger install option, as it comes with git (again Git for Windows) and the BASH tools all hooked up.  Some of the other features it brings along are portability (which is great if you are running events), keyboard shortcuts like copy and paste, easy aliasing, and more.

You install it by downloading the "Full" option on the [Cmder page](http://cmder.net/).  The video walk-through is at [todo](https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows).

### Pro's

- portable 
- brings copy/paste, resizing and more options to command prompt in older versions of Windows 
- nice looking prompt

### Con's 
- emulator (no quick shortcuts to open without other tools, *can be* less responsive)

## Your Favorite Git GUI
I don't want to get into the battle of the best GUI for git as they all have pros and cons.  The key here is that all of them come with git installed.  If you a fan of GUI's or just getting started this is great place to get started as you don't have to worry about the command line or setting up [command PATH's](https://technet.microsoft.com/en-us/library/bb490963.aspx?f=255&MSPPError=-2147217396).  

I tend install git via Chocolately then use a GUI for my day to day work. That said, I usually use the right tool for the right job when it comes to GUI vs command line. Some scenarios I really enjoy using the GUI are when viewing complex diff's or helping make atomic commits (I made more than one change because I got carried away and want to commit then one at a time ;-) ).  

I don't have a section on installing the GUI's in the video as there are to many to show, each one is behaves differently and are relatively easy to install and set up.

### Pro's

- easy guided git experience through GUI
- no need to mess with PATH and other tooling

### Con's 
- git not usually added to PATH (can't kick over to command line when 
- everyone prefers a different UI and may not know how to use yours

## Bash on Windows
This is one of the more promising solutions for the future.  At the time of writing, it is in preview on the Fast ring of the [Windows Insider program](https://insider.windows.com/).  I talk about why this is a promising solution on my [Thoughts on Build 2016](http://www.jamessturtevant.com/posts/Thoughts-on-Microsoft-Build-2016/) post along with a demo.  This allows us to run git *directly on BASH on Ubuntu on Windows*.  This means that we are not using a Windows implementation of BASH but the **real BASH**. More in-depth details on the from the developers themselves can be found at the discussion on [Linux Command Line on Windows](https://channel9.msdn.com/Events/Build/2016/C906).  This option is going to be really exciting for open source projects where developers might work on a variety of platforms. 

The set up for this requires you to be on the Windows Insider program at the time of writing.  Once you have the latest insider build installed you can follow the instructions at https://blogs.msdn.microsoft.com/commandline/2016/04/06/bash-on-ubuntu-on-windows-download-now-3/.  

Check out the video to see a walk-through [TODO](https://channel9.msdn.com/Blogs/jsturtevant/5-Way-to-Install-Git-on-Windows).

### Pro's

- real BASH and git
- it is the real BASH!
- fast

### Con's 
- still in preview
- requires the insider preview build

## Conclusion
There are many options for getting Git install on Windows to create a great experience no matter what your requirements are.  As with any choice there are trade off's depending on your specific scenario so try a few of the options out and find the one that works best for you.  Did I miss one?  What is your favorite?  Let me know in the comments below.