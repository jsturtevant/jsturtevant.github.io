---
layout: post
title: Using Azure Python SDK with Event Hubs
date: "2015-06-12 20:37"
categories:
  - azure
  - python
  - woodstove
  - IoT
---

[Python](https://www.python.org/) is a beautiful and powerful language.  Many times in just a few lines of code you can accomplish amazing things.  Following the python philosophy, the Azure team has provided an easy to use [Python API](https://github.com/Azure/azure-sdk-for-python) for the [Azure Service Bus](http://azure.microsoft.com/en-us/services/service-bus/).

But until recently it only worked with [Queues](https://azure.microsoft.com/en-us/documentation/articles/service-bus-dotnet-how-to-use-queues/) and [Topics](https://azure.microsoft.com/en-us/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/). Interacting with the [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/) using Python required us to write [pure rest calls](http://blog.kloud.com.au/2014/10/11/the-internet-of-things-with-arduino-azure-event-hubs-and-the-azure-python-sdk/), which works fine but is not as clean and simple as python libraries usually are.  But rejoice becuase this is no longer the case!

## Azure SDK with Event Hubs
After some digging for my [wood stove project]((https://github.com/jsturtevant/woodstove)), I found the latest release of the Python Azure SDK ([version 0.11 on May 13, 2015](https://github.com/Azure/azure-sdk-for-python/tree/v0.11.0)) now includes a simple SDK to send events to the Event Hub:

{% highlight python %}
from azure.servicebus import ServiceBusService

sbs = ServiceBusService(api_key["namespace"], shared_access_key_name=api_key["policy_name"], shared_access_key_value=api_key["policy_secret"])

sbs.send_event('eventhubname', '{ "DeviceId":"deviceId1", "Temperature":"10
.0" }')
{% endhighlight %}

## Conclusion
In just three lines of code we have enabled a python client to send data to an incredibly powerful Event Hubs ingestion service.  You can see a full [Internet of Things](http://www.jamessturtevant.com/posts/Abstractions-and-IoT/) (IoT) example using the Azure Python library at my [wood stove github project](https://github.com/jsturtevant/woodstove) and the documentation for the Azure Python API is found at [http://azure-sdk-for-python.readthedocs.org/en/latest/servicebus.html#event-hub](http://azure-sdk-for-python.readthedocs.org/en/latest/servicebus.html#event-hub).
