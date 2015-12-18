---
layout: post
title: ' Compiling ASP.NET 5 RC1 on a Mac or Linux'
date: '2015-12-02 15:04'
categories:
  - asp.net
  - csharp
---
<p class="message">Update (12/18/2015):  It is possible to compile ASP.NET 5 for the full .NET Framework on a Mac or Linux machine using <a href="http://www.mono-project.com/">Mono</a>.  More information can be found under the <a href="https://docs.asp.net/en/latest/getting-started/installing-on-mac.html">instructions for installing DNX for Mono on Mac. </a>
</p>

It is very exciting times as the [Release Candidate for ASP.NET 5](https://get.asp.net/) was released a couple weeks ago.  Of course, I just had to [try it out on my Mac](https://docs.asp.net/en/latest/tutorials/your-first-mac-aspnet.html) and it works!  Simply awesome!  I think this really opens up the doors for a lot of developers to try out ASP.NET and C#.

## The Build Issue
I did run into one problem with compiling the project on my Mac which slowed me down when I first started the project.  After creating a project using the [ASP.NET Yeoman Generator] (https://github.com/OmniSharp/generator-aspnet) (```yo aspnet```), I tried to build the project (```dnu build```) and got the following compile errors:

{% highlight bash %}
Building bosccdotnet for DNX,Version=v4.5.1
  Using Project dependency bosccdotnet 1.0.0
    Source: /Users/jsturtevant/projects/aspnetbostoncodecamp/src/bosccdotnet/project.json

    Unable to resolve dependency EntityFramework.Commands 7.0.0-rc1-final

    Unable to resolve dependency EntityFramework.MicrosoftSqlServer 7.0.0-rc1-final

    //removed for brevity

    /Users/jsturtevant/projects/aspnetbostoncodecamp/src/bosccdotnet/project.json(9,33): error NU1001: The dependency EntityFramework.Commands >= 7.0.0-rc1-final could not be resolved.

    /Users/jsturtevant/projects/aspnetbostoncodecamp/src/bosccdotnet/Startup.cs(9,7): DNX,Version=v4.5.1 error CS0246: The type or namespace name 'Microsoft' could not be found (are you missing a using directive or an assembly reference?)

    /Users/jsturtevant/projects/aspnetbostoncodecamp/src/bosccdotnet/Models/ApplicationDbContext.cs(4,7): DNX,Version=v4.5.1 error CS0246: The type or namespace name 'System' could not be found (are you missing a using directive or an assembly reference?)

    /Users/jsturtevant/projects/aspnetbostoncodecamp/src/bosccdotnet/Controllers/AccountController.cs(461,24): DNX,Version=v4.5.1 error CS0103: The name 'RedirectToAction' does not exist in the current context

    //removed for brevity
{% endhighlight %}  

## The Fix
The issue is obvious from the first line in the error message if you are aware of how ASP.NET 5 is architected but for the uninitiated this could be confusing. The issue is that you are building for the full .NET framework which is not available on Mac or Linux when using .NET Core based DNX (you can switch your .NET Execution Environment (DNX) using the .NET Version Manager (DNVM) and use [Mono](http://www.mono-project.com/)).  The issue is well known and is documented in the [Release Notes for RC1](https://github.com/aspnet/home/releases/v1.0.0-rc1-final).  Note that you can still work on the project on a Mac even with the errors as you are building for both versions of the .NET Framework.

The fix is also fairly straight forward.  The ```project.json``` has references for the .NET Frameworks that ASP.NET 5 can be compiled for:

{% highlight json %}
 "frameworks": {
    "dnx451": { },
    "dnxcore50": { }
  },
{% endhighlight %}  

As you can see, the default Yeoman Generator Template has ```dnx451``` and ```dnxcore50``` frameworks listed.  The ```DNX451```  reference is for the full .NET Framework which is not available on Mac or Linux.  The ```dnxcore50``` reference is a subset of the full .NET Framework that has been [open source and developed to be cross platform](https://github.com/dotnet/corefx).  The fix is as simple as removing the reference for ```dnx451``` in the ```project.json``` file  so that only the reference to ```dnxcore50``` is left:

{% highlight json %}
 "frameworks": {
    "dnxcore50": { }
  },
{% endhighlight %}  

Now you should be able to run ```dnu build`` again and have a successful build.  I hope this helps someone if they run into the same build errors.  Being able to compile and run on Mac and Linux is awesome.  I can't wait to see where the platform goes from here!
