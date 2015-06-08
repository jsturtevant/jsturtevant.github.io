---
layout: post
title: "A Wood Stove monitoring system built using ASP.NET MVC on Azure with Eventhubs, Stream Analytics and SignalR"
date: "2015-05-31"
categories:
- woodstove
---

## The problem
I live in Maine where wood stoves are plentiful.  The problem I encountered was when I would get [involved in my development ](http://www.jamessturtevant.com/posts/Music-and-The-Flow/) work, I would often let the fire burn out and the house would get cold (and more importantly my wife).  

In response, last year I built a prototype for a woodstove monitor, with [Floyd Hilton](http://floydhilton.com/).  The real-time monitoring system allowed me to sit in my office upstairs and see the temperature of my wood stove.  I was able to get reminders, and keep the house warmer for the family.  

## The plan
This year I want to take the system to the next level.  I would like to be able to get notification through different means (email, text, application alerts…), view the data in whatever context I am in (computer, mobile phone, smart watch) and have an easily deployable solution.
I will be building this system out over the next few months and would enjoy it if you came along for the ride.

There are a lot of [powerful tools](http://www.jamessturtevant.com/posts/Abstractions-and-IoT/) to enable IoT devices.  The main technologies I will be using are:

- [Eventhubs](http://azure.microsoft.com/en-us/services/event-hubs/) - Azure Event ingestion service that allows for millions of events per second
- [Stream Analytics](http://azure.microsoft.com/en-us/services/stream-analytics/) - Azure Real-time stream processing service similar to [Apache Storm](http://storm.apache.org/)
- [SignalR](http://signalr.net/) Real-time [websocket](https://developer.mozilla.org/en-US/docs/WebSockets) library that allows for bi-directional communication between applications.
- [ASP.NET MVC](http://www.asp.net/mvc) - C# Web application framework
- [RasberryPi](https://www.raspberrypi.org/) - Small inexpensive computer used for prototyping hardware solutions.

I am going to be trying out new types of deployment and hosting options.  I will also be trying out the new design craze called [Microservices]
(http://martinfowler.com/articles/microservices.html). I will be documenting my progress, success and failures so please be patient and understanding.

Check out the code repository at [https://github.com/jsturtevant/woodstove](https://github.com/jsturtevant/woodstove).
