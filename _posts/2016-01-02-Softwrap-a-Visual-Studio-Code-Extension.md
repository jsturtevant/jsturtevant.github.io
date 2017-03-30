---
layout: post
title: 'Softwrap: A Visual Studio Code Extension'
date: '2016-01-02'
categories:
  - vscode
  - extensions
---

> NOTE: This extention is no longer needed. Visual Studio now supports word wrap.  The defualt short cut is ```ALT -Z```

I use [Visual Studio](https://www.visualstudio.com/) for most of my development work but I have always had a second editor that was smaller, lightweight and fast, to help accomplish those quick one off edits.  A second editor always comes in handy when working on a configuration files, text files or maybe just peeking at code in another project.  

I have always used [NotePad++](https://notepad-plus-plus.org/) for quick edits because of the speed at which it opens files and  I currently use [Atom to write my blog posts](http://www.jamessturtevant.com/posts/atom-and-markdown-writer-for-jekyll/) as it has support for [Jekyll](http://jekyllrb.com/).  But, as a primarily .NET developer, I find that most of the editors don't have great out of the box support for C# syntax.  Sure I can set it up with great tooling like [Omnisharp](http://www.omnisharp.net/) but that's an extra thing I have to configure, even if it is [made easy with tools](http://www.jamessturtevant.com/posts/Chocolatey-And-Boxstarter/).

[Visual Studio Code](https://code.visualstudio.com/) was announced just last year and has had some great momentum as an editor and so I had to take a look at it for my quick editing experience.  So far I am very impressed with speed but I also get the syntax highlighting out of the box that I was missing from other editors.  There are some other great features it has borrowed from [Visual Studio](https://www.visualstudio.com/) that make it an attractive choice such as [IntelliSense](https://code.visualstudio.com/Docs/editor/editingevolved#_intellisense), [debugging](https://code.visualstudio.com/Docs/editor/debugging), and [Code Lens](https://code.visualstudio.com/Docs/editor/editingevolved#_reference-information). Not to mention it is also [open source](https://github.com/Microsoft/vscode/).  

## Soft wrap in VS Code
VS Code is still in beta so they are [adding features](https://code.visualstudio.com/Updates) all the time.  As I have started to use it more often, especially for editing text files and Markdown, I noticed that it was missing a ["soft wrap"](https://en.wikipedia.org/wiki/Line_wrap_and_word_wrap) feature for wrapping lines of text, which made it hard to read long lines without having to scroll. 

I looked around a bit but couldn't find any information on how to wrap the text lines. Even Google/Bing didn't come up with any results.  So I [reach out on twitter and got a response](https://twitter.com/Aspenwilder/status/677906524371599361) from [Errich Gamma](https://twitter.com/ErichGamma).  

To enable Soft wrap in VS Code you have to open your user settings file ([open command pallet](https://code.visualstudio.com/Docs/editor/codebasics#_command-palette), type 'settings', select 'Open User Settings') and add the following line:

{% highlight json %}
{
// there might be other custom settings you have in this file.
"editor.wrappingColumn": 0
}
{% endhighlight %}  

The default value is 300 and if you take a look at the default settings you will find the following comment which specifies setting the ```editor.wrappingColumn``` value to zero will cause a line wrap equivalent to a "soft wrap":

> // Controls after how many characters the editor will wrap to the next line. Setting this to 0 turns on viewport width wrapping

## Softwrap the Visual Studio Code Extension
Obviously, manually opening the User Setting file and editing the ```editor.wrappingColumn``` value was going to get old quickly.  So I did what any programmer would do... automate the process.  

The result is [Softwrap](https://marketplace.visualstudio.com/items/jsturtevant.softwrap), an extension for VS Code, that enable you to quickly switch back and for between a "soft wrap" and your more friendly code wrap.  Using [Softwrap](https://marketplace.visualstudio.com/items/jsturtevant.softwrap) is simple:

1. Open Command Pallet 
2. Type 'Softwrap' in the command window
3. Hit Enter.

That is it!  [Softwrap](https://marketplace.visualstudio.com/items/jsturtevant.softwrap) will toggle on/off and preserve the value it was previously set to even if you had a custom value for ```editor.wrappingColumn``` already set in your User Settings file.  You can check out the source at my [GitHub repository](https://github.com/jsturtevant/vscode-softwrap).

## Softwrap Implementation Notes
Overall I was really impressed with [VS Code's Extension system and documentation](https://code.visualstudio.com/docs/extensions/overview).  It was only announced about a month ago and yet it is fairly robust and detailed documentation.  

### Reading and Writing the User Settings File
The limitation I ran into when trying to write to the User Settings file was that there is no VS Code API for it [(see github issue)](https://github.com/Microsoft/vscode/issues/1396).  Again, I got a [quick response on twitter](https://twitter.com/ErichGamma/status/678667459621031936) that this was something that is under consideration but not yet implemented.  The suggestion was to directly modify the file itself as a work around.

This was fairly simple to do because VS Code is sitting on top of [Electron](http://electron.atom.io/) and we can leverage all the power and infrastructure of Node.js and the strong ecosystem that comes with it.  It was easy enough to use the [File System module](https://nodejs.org/api/fs.html) to read/write the User Settings file.

### Finding the Settings Files cross-platform
It was not easy to find the location of all the settings files accross each platform.  I finally found the [documentation here](https://code.visualstudio.com/docs/customization/userandworkspace#_settings-file-locations) and these two posts on how to work with Node.js cross-platform:

- [Cross-platform Node.js](http://shapeshed.com/writing-cross-platform-node/?utm_content=buffer72a33&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer)
- [User Data Folder Location]( http://stackoverflow.com/a/26227660/697126) 

### Comments in User Settings File
VS Code uses a custom JSON schema that allows comments in the settings file.  My current implementation does not support comments and will strip them out.  This is probably not the best way to handle the situation but will have to do for now.

## Conclusion 
Visual Studio Code is a strong candidate for my go to editor when editing quick files. It was very quick to [build out an extension for VS Code](https://marketplace.visualstudio.com/items/jsturtevant.softwrap) and the community is there for support. 

Because it has some [advanced support for debugging, editing](https://code.visualstudio.com/Docs/editor/editingevolved), and has a robust extension system I am even considering it for my go to tool for [ASP.NET 5 development](https://docs.asp.net), certainly it is the best option for [Node.js development](https://code.visualstudio.com/Docs/runtimes/nodejs).  
