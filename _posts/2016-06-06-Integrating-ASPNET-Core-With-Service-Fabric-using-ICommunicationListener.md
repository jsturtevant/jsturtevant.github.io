---
layout: post
title: Integrating ASP.NET Core with Service Fabric using ICommunicationListener
date: "2016-06-06"
categories:
  - service fabric
  - aspnetcore
  - asp.net
---

At the time of writing there is no official template for integrating ASP.NET Core (RC2) with Service Fabric in a [Stateless Service](/posts/Service-Fabric-Service-Types/) so I though I would dive in and see if I could get it to work by looking at the ASP.NET 4.6 Web API Template that is currently available through Visual Studio.  There will be a ASP.NET Core template available eventually but I thought this would be a useful exercise to learn more about how Service Fabric integrates with frameworks like ASP.NET.  

You can find the full implementation of the solution I came up with on [GitHub](https://github.com/jsturtevant/AspNetCoreTemplates).  There were also a couple examples that I used for reference.  The first was a really [basic template on GitHub](https://github.com/weidazhao/AspNetCoreTemplates) which I used as my starting point to creating an ASP.NET Core Communication Listener.  The other was the more [advance solution](https://github.com/weidazhao/Hosting) that shows not only how to integrate but also build an API Gateway after the [Microservices Gateway pattern](http://microservices.io/patterns/apigateway.html).

## ICommunicationListener
Lets take a look at the Service Fabric's [```ICommunicationListener```](https://msdn.microsoft.com/en-us/library/microsoft.servicefabric.services.communication.runtime.icommunicationlistener.aspx?f=255&MSPPError=-2147217396) interface which provides a few basic benefits.

First, it allows for service communication in an agnostic manner.  This means you can communicate with HTTP, UDP, or another other custom protocol you might need for your application by implementing this interface.  You can even communication with more than one protocol for one service.

Another benefit it offers is allowing for your service to be moved without affecting any clients that are working with it.  Any given service can move to another server because of machine failure, upgrade, resource balancing, etc., causing a change in the IP address of the actual service. By implementing this interface you are saved from having to manage any of those situations, instead Service Fabric manages it for you.  

The ```ICommunicationListener``` interface accomplishes this by registering your service with the Service Fabric Naming Service which will then keep track of the endpoint locations where the services are located.  This means a client can ask the Naming Service for a service by name and will be given the correct address without the need to know which machine it is actually running on (think along the same lines as DNS).  

It achieves this through a simple interface ```ICommunicationListener``` which has three methods:

- **Abort** - exit immediately, cancelling current requests
- **CloseAsync** - stop taking any new requests, close current requests gracefully and then exit
- **OpenAsync** - Open listener to be able to accept and send messages

As you can see it is a fairly simple interface but is a power concept because of the ability to allow for any type of communication and even multiple protocols per service.

## Building an ASP.NET Core Communication Listener
The first method we will implement is the ```OpenAsync``` Method.  In the case of the ASP.NET Core Communication Listener, this method will be responsible for starting the web service and binding it to the correct URL.  But first we need to gather some information about where the service is running from Service Fabric.

### Gathering Service Fabric Information
Since our service could be running on any server inside our cluster we need to get some of this information from Service Fabric runtime.  We use the ```FabricRuntime``` class to get the ```CodePackageActivationContext``` and ```NodeContext``` which contains information like the current service protocol, current IPAddress, and port.  We can get this information and format it appropriately by doing the following:

```csharp
var endpoint = FabricRuntime.GetActivationContext().GetEndpoint(endpointName);

string serverUrl = $"{endpoint.Protocol}://{FabricRuntime.GetNodeContext().IPAddressOrFQDN}:{endpoint.Port}";
```

The ```endpointName``` variable is being passed in from the initial creation of the service in the ```program.cs``` file.  It is any arbitrary name you decide to give the service.  Take a look at how this all come together in the full solution on the [GitHub repository](https://github.com/jsturtevant/AspNetCoreTemplates). 

### Starting ASP.NET 
At it's heart ASP.NET Core is actually a command line console application.  To start an ASP.NET application we use the ```WebHostBuilder``` class.  You can find an implementation of this in the ASP.NET Core RC2 template that ships with Visual Studio.   We are going to use slightly different setup than that template; The main differences are that we don't need to use IIS Integration and we need to specify the ```serverUrl``` that we just generated with the Service Fabric runtime information:

```csharp 
var host = new WebHostBuilder()
                .UseKestrel()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseUrls(serverUrl)
                .Build();

host.Run();
```

### Implementing Open
Now that we know have the current URL of the service and know how to start the ASP.NET server, we can finish our implementation of the ```OpenAsync``` method.  

This is just bringing together the previous two pieces and returning a server Url as a string.  The return value of the ```OpenAsync``` method will be the address that is registered with the Service Fabric Naming Service allowing for the service to move around in the case of machine failure, upgrades, etc.

Here is what the whole ```OpenAsync``` Method looks like:

```csharp
 public Task<string> OpenAsync(CancellationToken cancellationToken)
{
    var endpoint = FabricRuntime.GetActivationContext().GetEndpoint(endpointName);
    string serverUrl = $"{endpoint.Protocol}://{FabricRuntime.GetNodeContext().IPAddressOrFQDN}:{endpoint.Port}";


    webHost = new WebHostBuilder
                    .UseKestrel()
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseStartup<Startup>()
                    .UseUrls(serverUrl)
                    .Build();

    webHost.Start();

    return Task.FromResult(serverUrl);
}
```

### Implementing Abort and Close
We still need to implement two more methods on the ```ICommunicationListener```   before we are finished: ```Abort``` and ```CloseAsync```.  These are fairly simple implementations and just require us to dispose of the ```WebHost``` object. You could get fancier and make sure all requests are handled but for right now we will just shut down the web service as that is what is done in the ASP.NET Web API template currently.

```csharp
public void Abort()
{
    if (webHost != null)
    {
        webHost.Dispose();
    }

}

public Task CloseAsync(CancellationToken cancellationToken)
{
    if (webHost != null)
    {
        webHost.Dispose();
    }

    return Task.FromResult(true);
}
```


### Register the Asp.Net Core Communication Listener with Service Fabric
We have created the Asp.Net Core Communication Listener but still need to register it with the ```StatelessService``` that will be using ASP.NET.  There is a method on the ```StatelessSerivice``` Class that you can overload to return make the connection.  

This is the integration point between Service Fabric and Asp.Net.  In the case of Stateless service this returns the an array of ```ServiceInstanaceListners```. 

In the Stateless Service class, override the ```CreateServiceInstanceListeners```:

```csharp
internal sealed class WebHostingService : StatelessService
{
    private readonly string _endpointName;

    public WebHostingService(StatelessServiceContext serviceContext, string endpointName)
        : base(serviceContext)
    {
        _endpointName = endpointName;
    }

    protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
    {
        return new[] { new ServiceInstanceListener(serviceContext => new AspnetCoreCommunicationListener(_endpointName)) };
    }
}

````

## Next steps   
This is a simple implementation of a Communication Listener for ASP.NET Core.   If you would like to see the full working sample, check out [this GitHub repository](https://github.com/jsturtevant/AspNetCoreTemplates).  I have implemented everything we talked about but also started to add in the ability to unit test the ```AspNetCoreCommunicationListener```. 

Eventually there will be a template that Visual Studio will ship with that will provide a default implementation.  The defualt will be a likely good choice for getting started but this exercise of working through how to implement a ```ICommunicationListner``` is a good introduction to the ```ICommunicationListener``` interface and helps us better understand another aspect of the Service Fabric platform.
