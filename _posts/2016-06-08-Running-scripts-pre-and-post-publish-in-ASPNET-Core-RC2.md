---
layout: post
title: Running Scripts Pre and Post Publish in ASP.NET Core RC2
date: "2016-06-08"
categories:
  - aspnetcore
  - asp.net
---

This week I ran into an issue when deploying a Service Fabric service where a library [file was not being copied to the output directory]().  Since I am using [ASP.NET Core for my API layer](/posts/Integrating-ASPNET-Core-With-Service-Fabric-using-ICommunicationListener/) in the Service Fabric project, I had to go dig around to figure out how to get the file to copy to the deployment folder during the [publishing of the ASP.NET Core application](https://docs.asp.net/en/latest/publishing/web-publishing-vs.html).  

> note that this solution with project.json may change with as [ASP.NET Core moves back to .csproj file](https://blogs.msdn.microsoft.com/dotnet/2016/05/23/changes-to-project-json/)

What I learned is that in ASP.NET Core RC 2 you can add scripts to the ```prepublish``` and ```postpublish``` events by adding a ```scripts``` section to the ```project.json``` file.  In my case I needed to copy a file to the deployment folder after the rest of the project as built.

```json
{
  "buildOptions": {
       // ...
    }
  },

  //...

  "scripts": {
    "prepublish": ["<command1>, <command2>"],
    "postpublish": [ "xcopy <pathtofile> %publish:OutputPath%" ]
  }
}
```

Note that you can reference variables like ```%publish:OutputPath%```.  You can also supply multiple commands by separating them with a comma.  As noted above this currently works in ASP.NET Core RC2 but might change when as we [transition back to the .csproj file](https://blogs.msdn.microsoft.com/dotnet/2016/05/23/changes-to-project-json/).
