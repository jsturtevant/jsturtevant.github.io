---
layout: fwtpost
title: ".NET Secure String"
date: "2015-03-09"
categories:
- csharp
- .net framework tour
---


Today we are going to take a look in the System.Security namespace to learn how to create secure strings in .NET Framework.  When the standard string is created in your application it is stored in memory and there is no way to control when it is destroyed.  This is not ideal for applications that need to work with sensitive information such as Social Security Numbers or Credit Card numbers.  If your application works with this type of data it might be worth taking some time to evaluate the SecureString class.

The SecureString class will automatically encrypt the data when it is stored in memory.   It also implements the IDisposable interface so that you can control when the string is destroyed.  Even better than just freeing memory, the Dispose method writes binary zeros to the allocated memory before it is freed.  This ensures that the string is not readable after it's memory is done being used.

Another feature is that the string created is mutable until you tell the string to be read-only.  This means it can be manipulated over time, where as the standard string class is immutable.  Since it is not related to the standard string class it also does not have some of the same methods to compare or convert the string.  The documentation states this is to help prevent any accidental/malicious exposure.

To use it:

{% highlight csharp %}
var secureString = new SecureString();
secureString.AppendChar('s');
secureString.AppendChar('e');
secureString.AppendChar('c');
secureString.AppendChar('u');
secureString.AppendChar('r');
secureString.AppendChar('e');

// test to compare strings.
{% endhighlight %}

## More Information
You can find out more information on the [MSDN SecureString page](https://msdn.microsoft.com/en-us/library/system.security.securestring%28v=vs.110%29.aspx).   You can also find runnable examples on my .NET Framework Tour GitHub [repository.](https://github.com/jsturtevant/DotNetTour)
