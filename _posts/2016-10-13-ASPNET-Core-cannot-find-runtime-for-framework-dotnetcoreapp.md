---
layout: post
title: ASP.NET Core cannot find runtime for Framework '.NETCoreApp'
date: "2016-10-13"
categories:
  - aspnetcore
  - asp.net
---

Today I upgrade my ASP.NET Core application from version 1.0 to version 1.0.1 because of a [bug in Entity Framework](https://github.com/aspnet/EntityFramework/issues/5454).   Right after updating to the latest ASP.NET Core version, I built the project but ended up with the following error in Visual Studio:

```
Can not find runtime target for framework '.NETFramework,Version=v4.5.1' compatible with one of the target runtimes: 'win10-x64, win81-x64, win8-x64, win7-x64'. Possible causes:
The project has not been restored or restore failed - run dotnet restore
You may be trying to publish a library, which is not supported. Use dotnet pack to distribute libraries.
```

## Fix - Update project.json
After searching around for a few minutes I found [issue #2442 on GitHub](https://github.com/dotnet/cli/issues/2442#issuecomment-233154291).  This the issue states that you need to update your ```project.json``` and you have two options:

(1). Include the platforms you want to build for explicitly:

```json
"runtimes": {
    "win10-x64": {},
    "win8-x64": {} 
},
```

(2). Update the reference ```Microsoft.NETCore.App``` to include the ```type``` as ```platform```:

```json
"Microsoft.NETCore.App": {
    "version": "1.0.1",
    "type": "platform"
}
```

## Conclusion 
For more information on .NET Core Application Deployment you can [read the docs](https://docs.microsoft.com/en-us/dotnet/articles/core/deploying/index).  This is again another reason I love that all this work is being done out in the open. It really makes finding issues and bugs of this type easier.  Hope that Helps!