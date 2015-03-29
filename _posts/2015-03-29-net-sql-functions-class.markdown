---
layout: post
title: .NET SQL Functions Class
date: "2015-03-29"
categories:
  - csharp
  - .net framework tour
  - Entity Framework
---

If you use [Entity Framework](http://www.asp.net/entity-framework) you will enjoy learning about the [SQL Functions Class](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions(v=vs.110).aspx).  The SQL function class provides a way to call common database functions in your Entity Framework LINQ queries.

There are too many functions provided by this class to cover them all.  You will find some classic SQL functions like [Stuff](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions.stuff(v=vs.110).aspx) and [GetDate](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions.getdate(v=vs.110).aspx).  You will also find mathematical functions such as [PI](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions.pi(v=vs.110).aspx) and [Log](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions.log(v=vs.110).aspx).

It is important to note that these functions can only be called within the context of the [LINQ to Entity queries](https://msdn.microsoft.com/en-us/library/vstudio/bb386964(v=vs.100).aspx).  An example of how you would use these functions is as follows:

{% highlight csharp %}
var results = _testContext.Persons
.Where(x => x.CreateDate >= SqlFunctions.GetDate())
.ToList();

Assert.AreEqual(1, results.Count);
Assert.AreEqual("Sam", results.First().Name);
{% endhighlight %}

## More Information
You can explore the many functions provided by the [SQLFunctions class](https://msdn.microsoft.com/en-us/library/system.data.objects.sqlclient.sqlfunctions(v=vs.110).aspx) on MSDN.  This class will not be avaliable in the [.NET Core project](https://github.com/dotnet/corefx).  To see a list of libraries that will be avalible in the .NET Core project check out the [corefx-progress GitHub repository](https://github.com/dotnet/corefx-progress).
