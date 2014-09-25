---
layout: post
title:  "Visual Studio Extensions-Ace-Jump"
date:   2014-09-25
categories: Visual Studio
---

This week I released my first Visual Studio Extension [AceJump](http://visualstudiogallery.msdn.microsoft.com/2d045428-ec7e-4a77-802c-5365f9ddafa2).  I was inspired by a version of AceJump build by [John Lindquist](http://plugins.jetbrains.com/plugin/7086?pr=phpStorm) for the JetBrain tools.  I first saw him use it in his tutorial video's on [egghead.io](https://egghead.io) and after installing it on [JetBrains Webstorm](http://www.jetbrains.com/webstorm/) I knew I needed to have the same functionality in Visual Studio.  I looked around the plugin's in Visual Studio and didn't find anything, so I built it.

## Visual Studio AceJump
As of this blog post it is a fully functional and has a few rough edges but  I decided to [ship it anyways](http://blog.codinghorror.com/version-1-sucks-but-ship-it-anyway/).  Getting software out to real users is important for the feedback loop.  Besides I wanted to start using it myself because it such a cool productivity tool.

It was my first Visual Studio extension so I floundered around in the SDK a bit.  It was quite the learning experience and I not sure I did everything the *'correct way'*.  I found once you get beyond [displaying a purple square](http://msdn.microsoft.com/en-us/library/dd885474.aspx) in the top right corner of the editor things get tougher.  You can maybe learn from something alittle more complex or show me a better way by taking a look at the [code for AceJump](https://github.com/jsturtevant/ace-jump).

## Usage
It is a really simple to use:

1. Use shortcut ```Ctrl + Alt + ;``` to display a key selector.  
2. Press any letter to highlight occurrences of that letter in the text editor.   
3. Press Character in the box that is over the letter you want to jump too.
4. Your cursor jumps to that spot and you keep coding!

I hope you find this tool as useful as I did when I first used it in Webstorm.  Happy Coding!