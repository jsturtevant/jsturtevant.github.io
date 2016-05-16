---
layout: post
title: Service Fabric Service Types
date: "2016-05-09"
categories:
  - service fabric
---

The model for [Service Fabric](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-overview/) allows for flexibility when it comes to designing the individual services, giving you the ability to pick the right programming model for each service.  Deciphering the multiple options can be complex and really depends on each scenario and service you are building.  Here I will try to break down the options how I see them.  Please leave feedback if it was helpful or you think of options in a different way.  Remember every application is composed of many services and each service will likely have a different programming model so assess each service individually.

## Service Types
There are two main types of services you can build with Service Fabric:

- **[Stateless Services](#stateless-services)** - no state is maintained in the service.  Longer term state is stored in an external database.  This is your typical application/data layer approach to building services that you are already likely familiar with.
- **[Stateful Services](#stateful-services)** - state is stored with the service.  Allows for state to be persisted with out the need for an external database.  Data is co-located with the code that is running the service.

One of the real advantage of using a platform like Service Fabric is when you begin to leverage the power of Stateful services.  But Stateful services takes a bit of a mind shift in the way we think about building applications so let's look at stateless applications first then move onto the stateful service approach. 

Once you have decided whether you are building a Stateful or Stateless service you can then choose from [multiple types of programming models](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-choose-framework/) to help you build you services:

- **Guest** - allows you to run any executable inside Service Fabric runtime.  This can be an executable or a container
- **Reliable Services** - light weight framework that lets you integrate with the Service Fabric runtime (can be stateful or stateless)
- **Reliable Actor** - framework that lets you build your applications using the Actor pattern (can be stateful or stateless)

## Stateless Services
Stateless services are a familiar paradigm in most web development.  You typically have a middle tier  layer that receives requests and then calls out to a database where the state is stored.  When those requests to the data store are expensive there is often a caching mechanism in place.  Since there is no state maintained inside the service a request can be routed to any instance of the service. A typical application using stateless services looks like:

![stateless application]({{ site.url }}/assets/stateless.png)
[modified image from https://github.com/Azure/azure-content/blob/master/articles/service-fabric/media/service-fabric-application-scenarios/AppwithStatelessServices.jpg]

It is important to note that the above diagram only uses stateless services.  Inside a service fabric application you will typically have a combination of stateless and stateful services.

### Stateless Application Use Cases
Typical use cases for a Stateless Application are:

- web interface for end users
- API gateway to other services
- proxies
- existing stateless service
- calculations that don't require state  (complex math calculations where all state is provided and a result is returned)

### Stateless Service Fabric Programming Models
The Service Fabric Programming model that you might use depends on the specific use case.  A few guiding tips:

- **Guest Executable** - There are a few scenarios where using an existing application make sense.  Existing applications that are moving over to Service Fabric is a great use case.  Another use case would be team familiarity with a language or the particular external service (data store or service) that is part of the system has better first class support for a language that doesn't have an Service Fabric Runtime API currently.  With a Guest Executable you get the advantages like service orchestration and rolling upgrades  but miss out on being able to tie into the more advance features that service fabric platform has to offer such as custom health reporting.
- **Stateless Reliable Services** - Use Stateless Reliable Services when you are building new services from scratch and want to take advantage of the Service Fabric platform features.  By using the Reliable Services API's you get access to the features like health monitoring, endpoint registration, load reporting and more.  Any application endpoints built using [ASP.NET Core](https://get.asp.net/) (MVC or WebAPI) are great use cases for Reliable Services. A [Java Reliable Services API](https://blogs.msdn.microsoft.com/azureservicefabric/2016/03/31/announcing-service-fabric-ga-on-azure-public-preview-of-standalone-clusters-on-windows-server-and-limited-preview-on-linux/) is in the works.
  
## Stateful Services
Stateful services are different from Stateless services in the fact that the data is stored on the same machine that the service is running.  *There is no external data store*.  This removes the need for additional layers like caches and queues, simplify the service. The latency in the system drops because the data can be retrieved without additional network calls to the external store.  A diagram says it all (be sure to compare this to the diagram of a stateless application):

![stateful application]({{ site.url }}/assets/stateful.png)
[modified image from https://github.com/Azure/azure-content/blob/master/articles/service-fabric/media/service-fabric-application-scenarios/AppwithStatefulServices.jpg]

Now that the data is on the same machine as the service, if the machine failed then you would lose all the data *but this is not the case with Service Fabric*. The Service Fabric runtime manages the reliability of the state for you so you do not have to worry about machine failures.  The way the reliability is accomplished using replicas is fairly complex and ultimately you don't have to think about it to much but I do recommend that you learn how the reliability of your data is ensured so you can understand some of the considerations you need to take in place.  A great introduction is in the video [Building MicroServices with Service Fabric](https://channel9.msdn.com/events/Build/2016/T693).

Another reason the Stateful approach to services is powerful is because now you remove the need to translate between your conceptual model and data model (multiple rows and tables in the case of a SQL database).  This simplifies your programming model and enables you to work in one mental model with out worry (to much) about how it is stored in the database.  This is achieved through the [Reliable Collections](https://azure.microsoft.com/en-us/documentation/articles/service-fabric-reliable-services-reliable-collections/) ```IReliableQueue``` and ```IReliableDictionary``` interfaces.

### Use Cases
Typical use cases for a Stateful application are:

- any data service (such as order service or inventory service)
- gaming scenarios
- most services where data is stored externally and pulled into do processing (you would model it so the data is local)
- data analytics and workflows

### Stateful Service Fabric Programming Models
The Service Fabric Programming model that you might use depends on the specific use case.  A few guiding tips:

- **Stateful Reliable Services** - allows the most flexibility around the managing state.  It allows you create transactions around multiple data types or create complex processing units of work.  Most Stateful services can be created using this programming model and is a great place to start if you are sure your domain doesn't fit the Actor Model.
- **Stateful Actor Model** - This model is useful anytime the [Actor pattern](https://en.wikipedia.org/wiki/Actor_model) can be used to describe  your domain space.  This sounds obvious but is only useful if you know where you might use the Actor model to model your domain space.  A few scenario's would be where you are doing a high number of small, independent calculations or have many concurrent interactions that need supervision.  Understanding the Actor model is outside the scope of this post. If you are not familiar with the Actor model I would recommend learning a little bit more so you can be sure to leverage this powerful programming model inside Service Fabric when appropriate. Another important note is the the level of reliability at which a Actor stores state can be configured.  They can be configured to store the state as persisted, volatile, and only in memory.

## Conclusion
There are two main types of services that can run inside Service Fabric. The first being Stateless services where state is stored in external service.  This is the familiar way to build services and applications today.  In the stateless applications you can either bring your own service or build one using the Service Fabric API's that let you tie into the Service Fabric runtime.  Tying into the runtime allows you to begin to leverage the platform to get more insight into your services.

The second type is a Stateful service where the data is co-located with code that runs the service reducing latency and reducing service complexity. There are currently two models for building Stateful services: Reliable Service and Reliable Actors.  By choosing a Stateful service you are beginning to tap into the Service Fabric platform which goes beyond service orchestration.

Hope that was a helpful break down of the services.  Let me know in the comments or tell me how you view the options for building services in Service Fabric.
