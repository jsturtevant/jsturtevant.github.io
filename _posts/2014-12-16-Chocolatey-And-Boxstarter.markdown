---
layout: post
title:  "Chocolatey and Boxstarter"
date:   2014-12-16
categories: fun, windows
---

Setting up a new computer is never fun.  That has all changed with some great tools called [Chocolatey](https://chocolatey.org) and [Boxstarter](http://boxstarter.org).  Long gone are the days of searching from site to site for the *download here* link.  No longer will you sit there clicking through installers.  Never again will you forget that to install the seldom used but critical program.

## Chocolatey
[Chocolatey](https://chocolatey.org) is tool to install applications and tools quickly with out the fuss.  From Chocolately's [about page](https://chocolatey.org/about):

> Chocolatey is a package manager for Windows (like apt-get or yum but for Windows). It was designed to be a decentralized framework for quickly installing applications and tools that you need. It is built on the NuGet infrastructure currently using PowerShell as its focus for delivering packages from the distros to your door, err computer.

Once installed it is simple to use, just open the command prompt and type:

```powershell
choco install git
```

The above command installs the latest version of [Git for Windows](http://msysgit.github.io/).  *And* you don't have to click through the installer.

## Boxstarter
[Boxstarter](http://boxstarter.org) lets you automate multiple Chocolately installs so you don't have to worry about required reboots.  It also lets you configure [Windows system settings](http://boxstarter.org/WinConfig) such as always [showing hidden file/folders](http://windows.microsoft.com/en-us/windows/show-hidden-files#show-hidden-files=windows-7).  From the [Why Boxstarter](http://boxstarter.org/WhyBoxstarter) page:

>Boxstarter leverages Chocolatey packages to automate the installation of software and create repeatable, scripted Windows environments.

The best part is that you can do it all with [one simple url](http://boxstarter.org/WebLauncher) and nothing installed on the machine to start.  Simply use following URL (seems to only work in IE):

```
http://boxstarter.org/package/sysinternals,fiddler4,itunes
```

The above url will prompt you for a download.  When you accept the download, the downloaded installer will automatically download Chocolatey and install the chocolately packages for [sysinternals](http://technet.microsoft.com/en-us/sysinternals/bb545021.aspx), [fiddler4](http://www.telerik.com/fiddler) and [itunes](https://www.apple.com/itunes/).  All with out any prerequisites!

## Putting it together
You can also create Boxstarter scripts and install then using a URL also.  I have created a GitHub repository called [Chocolatey-recipe](https://github.com/jsturtevant/chocolatey-recipe) to demonstrate how it would work.  To use it:

1. Download and run [RunBoxstarterInstall.bat](https://github.com/jsturtevant/chocolatey-recipe/blob/master/RunBoxstarterInstall.bat) (only works with IE as default browser but since this for a fresh Windows computer that is ok)
2. This will launch a download for an executable.
3. Run the executable it will download, install, and configure all programs in [chocolatey-recipe.txt](https://github.com/jsturtevant/chocolatey-recipe/blob/master/chocolatey-recipe.txt)

The [RunBoxstarterInstall.bat](https://github.com/jsturtevant/chocolatey-recipe/blob/master/RunBoxstarterInstall.bat) starts your browser with the following URL:

```
http://boxstarter.org/package/url?https://raw.githubusercontent.com/jsturtevant/chocolatey-recipe/master/chocolatey-recipe.txt 
```

Instead of passing package names, this time we pass the URL of a plain text Boxstarter script that contains 20+ of my favorite programs, installs windows updates and sets Windows explorer options.  Again, all *without* installing anything first.  Now come back a couple hours later and your machine is ready with everything installed!

Do you use Chocolatey and Boxstarter?  What other tools do you use?


