---
layout: post
title: Dependency Injection with Service Fabric
date: "2016-05-16"
categories:
  - service fabric
---

In my previous post I talked about how [Azure Service Fabric went to General Availability in Azure](/posts/Thoughts-on-Microsoft-Build-2016/) and why it was important.  Since then I have been diving deeper on Service Fabric and I will be giving a talk on it at [ContainerDays Boston 2016](http://dynamicinfradays.org/events/2016-boston/) next week.  Today we are going to talk about how to wire up dependency injection in Service Fabric for Reliable Services and Actors. This will be useful to understand so that later we can setup Unit Tests.

## Registering a Service 
First lets take a look at how you register a Reliable Service.  There are a couple different types of Reliable services: Stateless and Stateful.  For this post we don't need to know much about the differences but you can learn more at the post [Service Fabric Service Types](/posts/Service-Fabric-Service-Types/).   Both versions have a ```ServiceContext``` that is passed in to the constructor.  The default Visual Studio templates for stateless or stateful services actually has the basic Dependency injection set up in the ```program.cs``` file:

```csharp
ServiceRuntime.RegisterServiceAsync("State1Type", context => new Service(context)).GetAwaiter().GetResult();
```

Depending on the service type (stateful or stateless) the ```context``` that is passed into the service will either be a ```StatefulServiceContext``` or a ```StatelessServiceContext```.  Both derive from ```ServiceContect```.

As you can see we are already injecting our dependency on the ```ServiceContext``` into the service. Adding another dependency is easy:

```csharp
ServiceRuntime.RegisterServiceAsync("State1Type", context => new Service(context, new MyNewDependency())).GetAwaiter().GetResult();
```

For Stateless services that is all we need to do.  When it comes to Stateful services, if we want to be able to unit test the service, we also need to be inject the ```ReliableStateManager```:

```csharp
ServiceRuntime.RegisterServiceAsync("Stateful1Type", context => new Service(context, new ReliableStateManager(context), new MyNewDependency())).GetAwaiter().GetResult();
```

We then need to update the Stateful service to pass this dependency on to the base class in the Service constructor:

```csharp
public Service(StatefulServiceContext serviceContext,
            IReliableStateManager stateManager, IMyDependency dep)
            : base(serviceContext, stateManager as IReliableStateManagerReplica)
        {
            this._dep = dep;
        }
```

In later posts I will go into how to actually set up the unit tests for the services.  

## Registering an Actor
The Actor programming model is built on top of the [Reliable Services Model](/posts/Service-Fabric-Service-Types#stateful-service-fabric-programming-models).  Registering an Actor is fairly straight forward:

```csharp
ActorRuntime.RegisterActorAsync<Actor2>().GetAwaiter().GetResult();
```

The ```RegisterActorAsync``` method actually takes a second overload that allows you to register the Actor via a factory method and this is where you would set up dependency injections:

```csharp
ActorRuntime.RegisterActorAsync<Actor2>(
    (context, actorType) => new ActorService(context, actorType, () => new Actor2(new MyDependency()))).GetAwaiter().GetResult();
```

## Taking it to the next level
This is how you would set up dependency injection at the program level.  If you wanted to use a more advanced DI container system you might take it to the next level by using a container as in [this example](http://stackoverflow.com/a/35900027/697126).  Beware that there are lots of nuances for the way the Reliable Services are created and removed and could cause some trouble if you are not careful.  

Stay tuned as be begin to look at setting up unit tests for the Service Fabric services.  