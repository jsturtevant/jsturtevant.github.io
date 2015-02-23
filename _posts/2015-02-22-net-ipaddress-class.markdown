---
layout: post
title: .Net IPAddress Class
date: "2015-02-22"
categories:
- csharp
- .net framework tour
---

This week on the .NET Framework Tour, we are going to take a brief look at the the [IPAddress Class](https://msdn.microsoft.com/en-us/library/System.Net.IPAddress(v=vs.110).aspx) which lives in the  ```System.Net``` namespace.  The ```IPAddress``` class helps a developer work with IP addresses.  It can parse [IPv4](http://en.wikipedia.org/wiki/IPv4) and [IPv6](http://en.wikipedia.org/wiki/IPv6_address) strings and returns an ```IPAddress``` object that give us helpful information.  The class also helps with the manipulation of IP addresses.

To parse an IP address:
{% highlight csharp %}
IPAddress address;

Assert.IsTrue(IPAddress.TryParse("10.2.12.123", out address));
Assert.IsFalse(IPAddress.TryParse("10.2.12.1234", out address));

IPAddress ipv6Address = IPAddress.Parse("2001:0db8:85a3:0000:0000:8a2e:0370:7334");
Assert.AreEqual(AddressFamily.InterNetworkV6 , ipv6Address.AddressFamily);
{% endhighlight %}


To convert an IPv4 address to IPv6 address (and vice versa):
{% highlight csharp %}
IPAddress ipv4Address = IPAddress.Parse("10.3.5.156");

IPAddress ipv6Address = ipv4Address.MapToIPv6();
Assert.AreEqual(AddressFamily.InterNetworkV6, ipv6Address.AddressFamily);
{% endhighlight %}

## More Info

The class can do a lot more than just parse strings so check out [MSDN](https://msdn.microsoft.com/en-us/library/System.Net.IPAddress(v=vs.110).aspx) for more information about the ```IPAddress``` class.  There is also a [samples project](https://github.com/jsturtevant/DotNetTour) on GitHub so you can play with the class to learn a bit more.

The class is schedule to come over to the open source [.NET Core Libraries](https://github.com/dotnet/corefx).  See all items currently planned to move to the new open source library [here](http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-components-postattachments/00-10-58-94-19/NetCore_5F00_OpenSourceUpdate.xlsx).

<p class="message">
The .NET Framework provides helpful libraries to make you job easier, but it is impossible to know them all.  Sign up for the newsletter that will give you reminders of useful libraries you may have forgotten and maybe a few new ones!
</p>
