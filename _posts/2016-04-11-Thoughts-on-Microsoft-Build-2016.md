---
layout: post
title: Thoughts on Microsoft Build 2016
date: "2016-04-11"
categories:
  - bash
  - windows
  - service fabric
---

For the last few years there has been a major shift at Microsoft, one that has culminated in news from [Build 2016](http://build.microsoft.com/) that I don't think anyone saw coming...

<iframe src="https://channel9.msdn.com/Blogs/jsturtevant/Thoughts-on-Microsoft-Build-2016/player" width="850" height="540" allowFullScreen frameBorder="0"></iframe>

## Bash on Windows
The most exciting announcement for me to come out of Microsoft Build is that [Bash Shell is coming to Windows 10](http://www.hanselman.com/blog/DevelopersCanRunBashShellAndUsermodeUbuntuLinuxBinariesOnWindows10.aspx)!  

If your just learning about this, stop everything and go [learn how to install Bash on Ubuntu on Windows](https://blogs.msdn.microsoft.com/commandline/2016/04/06/bash-on-ubuntu-on-windows-download-now-3/) right now.  Seriously this post will be there for you to read while you upgrade.

Bash on Ubuntu on Windows is a game changer in a couple different ways. First when a developer is working on the Windows platform wants to get stated with any given technology they no longer have to worry if it [is fully supported on Windows](https://twitter.com/avdi/status/713773013347512320). No longer will the developer have to [spend hours searching to the ends of the internet to figure out how to get it working for Windows](https://github.com/railsinstaller/railsinstaller-windows/issues/70).  This is great not only for Windows developers but is great for the languages and tools themselves because now they will now have an easier on boarding and learning process for a new audience.   On a deeper level this really means that there will be more collaboration and cross-pollination of ideas across the two eco systems.       

Second when teaching any open source technologies there was always the problem of having to give instructions for two platforms ([if the instructor bothered](https://twitter.com/avdi/status/713706439727058944)).  Personally I have felt this pain when running courses using Node.js and Python. I went to great lengths to make sure that I had [instructions for both Mac and Windows](https://github.com/jsturtevant/happy-image-tester-django) so that my students would have a great experience not matter what.  With Bash on Windows this simplifies the work I need to do as a teacher and more importantly makes the experience for my students a better one because I don't have to do the context switching.  Again this leads to an overall smoother experience for everyone!

You can find more details in this [video interview of the developers on the Bash in Window project](https://channel9.msdn.com/Events/Build/2016/C906).  More technical details are available from [Dustin from Ubuntu](http://blog.dustinkirkland.com/2016/03/ubuntu-on-windows.html).

## New members to the .NET Foundation
Another reason I am excited to be a .NET developer is because of the commitment and excitement around open source.  I think the announcement of Red Hat, JetBrains and Unity  joining the .NET Foundation really solidifies that the community is committed to .NET as an open and innovative platform.  JetBrains in particular for me has been one of the companies that really changed the way I developed and thought about tooling with the [ReSharper](https://www.jetbrains.com/resharper/) product. It is great to see them contributing to the direction of the .NET platform.

It really has been fun as an ASP.NET developer to watch the evolution of ASP.NET as it makes its journey cross-platform as ASP.NET Core.  There are lots of exciting things happening on that project but again I think the important thing to note is that this opens up a platform that was not accessible to a whole trove of developers;  developers that couldn't even try it out.  Now they can try .NET out which opens the door to new conversations and idea sharing.  That is the truly exciting part.

## Service Fabric goes GA
Microservices has been all the rage lately and [for good reason](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-overview-microservices/).  As you start to look into how to implement a microservices approach you will find there is a wealth of platform options to help you make sure you are delivering your services in a scalable, maintainable way.  Most provide a way for orchestration, monitoring and automation of your services but what if they provided more?

[Service Fabric](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-overview/) is one of the most innovative platforms for microservices.  It goes beyond just orchestration of your services and provides a platform that allows you to build your applications faster, with deeper diagnostics and health reporting.  It is one of the most exciting technologies I have seen come out of Microsoft and with the [Reliable Services application model](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-reliable-services-introduction/) it will change the way you think, develop and manage applications. 

The [announcement that Service Fabric is now Generally Available](https://azure.microsoft.com/en-us/blog/azure-service-fabric-is-ga/) means that we can start using the same technology that is used to build services like Cortana, Azure Event Hubs and Azure SQL Database.  Check out the [Azure Service Fabric for Developers](https://channel9.msdn.com/events/Build/2016/B874) video to get a feel for what is possible.  

The best part is that the platform it's self is free.  You can download the Service Fabric runtime and deploy it anywhere, even to AWS as they [demonstrate in this video]((https://channel9.msdn.com/events/Build/2016/B874)).  This is a huge change from the way Microsoft has approached services like this in the past. This is yet another reason why I am excited to be a .NET developer.  But this service is not only for .NET, they announced support [for integrating in Java with the reliable services and actors](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-linux-overview/) and you can run any executable (node.js/python/etc) as a service.

## Conclusion
All of this has led to companies looking at the Microsoft platform with new fresh look.  Companies are considering things that we never thought possible.  A great example would be Facebook bringing the Facebook Audience Network SDK to the Windows platform.  This is opening up new opportunities and experiences on both sides - enabling 3 million Facebook advertisers to reach Windows 10 customers.

I really am excited to be a .NET developer.  The ability to work side by side with developers from other platforms and languages excites me because of the possibilities of having conversations that were never possible before. There is opportunity to learn on both sides and I look forward to the next wave of innovations that is sure to come because of the new and open dialog we are having.

These were certainly not the only exciting announcements from Build 2016.  You can catch all the [keynotes and recorded technical sessions on Channel9](https://channel9.msdn.com/Events/Build/2016) and find more reactions from other Technical Evangelist's on [Jeremy Foster's Build 2016 Developer Reactions post](http://codefoster.com/build16react).  Be sure to let me know what you found most exciting in the comments below.   
