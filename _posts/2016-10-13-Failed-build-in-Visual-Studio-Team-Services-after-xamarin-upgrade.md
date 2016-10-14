---
layout: post
title: Failed Build in Visual Studio Team Services after Xamarin upgrade
date: "2016-10-14"
categories:
  - xamarin
  - Vsts
---

This week I updated Visual Studio's version of Xamarin.  All was well until I kicked off a build for my Andriod project in Visual Studio Team Services (VSTS).  During the ```Build Xamarin.Android``` Project step I recieved the following error and my build failed:

```
C:\Program Files (x86)\Java\jdk1.6.0_45\\bin\javac.exe -J-Dfile.encoding=UTF8 -d obj\Release\android\bin\classes -classpath ....
obj\Release\android\src\android\support\design\R.java:10: cannot access java.lang.Object
bad class file: java\lang\Object.class(java\lang:Object.class)
class file has wrong version 52.0, should be 50.0
Please remove or make sure it appears in the correct subdirectory of the classpath.
 public final class R {
```

The interesting part in the error message was the ```class file has wrong version 52.0, should be 50.0``` message.  This was ultimately what helped me find the error.  After a few minutes of searching, it was obvious that I had the wrong JDK version for building my assembly.  Most of the issues were Android Studio specific but I found an issue were this happened in Visual Studio.  This wasn't Visual Studio, instead VSTS but it got me down the right path.

## Fix - Setting the Build Version explicitly in Build.Andriod step
Now that I had some idea of what was going on I headed over to my VSTS Build process and noticed that there was an option in the ```Build Xamarin.Android``` step to set the JDK Version.  Looking at the output above I noticed I was using ```JDK 6```. I tried ```JDK 7``` and got the message ```Unsupported major.minor version 52.0```.  So I tried the only value left which was ```JDK 8``` and it worked.  Here is the screen shot of the updated setting:

![update jdk version number in vsts]({{ site.url }}/assets/vsts-xamarin-jdk-version.png)

I assume that there is a good explanation for the change but I have not found it yet.  Hope this helps!