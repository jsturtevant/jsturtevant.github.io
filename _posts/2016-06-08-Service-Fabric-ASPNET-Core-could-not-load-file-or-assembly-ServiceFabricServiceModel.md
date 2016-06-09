---
layout: post
title: Service Fabric ASP.NET Core could not load file or assembly ServiceFabricServiceModel
date: "2016-06-08"
categories:
  - service fabric
  - aspnetcore
  - asp.net
---

After adding a reference to retrieve a [Service Fabric Actor](/posts/Service-Fabric-Service-Types/) to my [ASP.NET Core Web API](/posts/Integrating-ASPNET-Core-With-Service-Fabric-using-ICommunicationListener/) project I got the following error when making the call to create the actor using ```IMyActor actor = ActorProxy.Create<IMyActor>(actorId, serviceuri);```:

```
System.IO.FileNotFoundException: Could not load file or assembly 'ServiceFabricServiceModel, Version=5.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies. The system cannot find the file specified.
File name: 'ServiceFabricServiceModel, Version=5.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
   at Microsoft.ServiceFabric.Services.Common.FabricServiceConfigSection.Initialize()
   at Microsoft.ServiceFabric.Services.Communication.FabricTransport.Common.FabricTransportSettings.LoadFrom(String sectionName, String filepath, String configPackageName)
   at Microsoft.ServiceFabric.Services.Communication.FabricTransport.Common.FabricTransportSettings.TryLoadFrom(String sectionName, FabricTransportSettings& settings, String filepath, String configPackageName)
   at Microsoft.ServiceFabric.Services.Communication.FabricTransport.Common.FabricTransportSettings.GetDefault(String sectionName)
   at Microsoft.ServiceFabric.Actors.Remoting.FabricTransport.FabricTransportActorRemotingProviderAttribute.CreateServiceRemotingClientFactory(IServiceRemotingCallbackClient callbackClient)
   at Microsoft.ServiceFabric.Actors.Client.ActorProxyFactory.CreateServiceRemotingClientFactory(Type actorInterfaceType)
   at Microsoft.ServiceFabric.Actors.Client.ActorProxyFactory.GetOrCreateServiceRemotingClientFactory(Type actorInterfaceType)
   at Microsoft.ServiceFabric.Actors.Client.ActorProxyFactory.CreateActorProxy[TActorInterface](Uri serviceUri, ActorId actorId, String listenerName)
   at Microsoft.ServiceFabric.Actors.Client.ActorProxy.Create[TActorInterface](ActorId actorId, Uri serviceUri, String listenerName)
   at Web.Controllers.Controller.<Create>d__1.MoveNext() in 
```

From [this stackoverflow answer](http://stackoverflow.com/a/37275624/697126), it seems that there is currently an issue with the dotnet cli when working with nuget packages that do not explicitly reference assemblies.   In this case, it is specifically an issue with [Microsoft.ServiceFabric.Services package (version 2.0.217)](https://www.nuget.org/packages/Microsoft.ServiceFabric.Services/2.0.217).  When the ASP.NET Core publish action occurs, the ```ServiceFabricServiceModel.dll``` does not get copied to the publish folder but it is in the packages folder for the library.

## Fixing with a postbuild script in project.json
This will likely be fixed but in the mean time you can fix this issue by adding a [postbuild script in the ASP.NET Core RC2](/posts/Running-scripts-pre-and-post-publish-in-ASPNET-Core-RC2/) ```project.json``` file.  I used ```xcopy``` to copy the file from the ```packages``` folder to the publish directory.  Once there, Service Fabric was able to package it up for deployment and everything worked.

In the ```project.json``` file add a scripts command:

```json
{
  //...

  "scripts": {
     "postpublish": [ "xcopy ..\\packages\\Microsoft.ServiceFabric.Services.2.0.217\\lib\\net45\\ServiceFabricServiceModel.dll %publish:OutputPath%" ]
  }
}
```

This will locate packages folder (maybe different for your setup) and copy it to the output folder for the publish command. 

