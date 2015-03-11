---
layout: fwtpost
title: ".NET Secure String"
date: "2015-03-09"
categories:
- csharp
- .net framework tour
---


Today we are going to take a look in the ```System.Security``` namespace to learn how to create secure strings in .NET Framework.  The standard .Net  [string](https://msdn.microsoft.com/en-us/library/362314fe.aspx) is stored in managed memory giving you no way to control when it is destroyed.  This is not ideal for applications that need to work with sensitive information such as Social Security Numbers or Credit Card numbers.  If your application works with this type of data it might be worth taking some time to evaluate the [SecureString](https://msdn.microsoft.com/en-us/library/system.security.securestring(v=vs.110).aspx) class.

The ```SecureString class``` will automatically encrypt the data when it is stored in memory.   It also implements the IDisposable interface so that you can control when the string is destroyed.  Even better than just freeing memory, the Dispose method writes binary zeros to the allocated memory before it is freed.  This ensures that the string is not readable after it's memory is done being used.

Another feature of ```SecureString``` is the string created is mutable until you tell the string to be read-only.  This means it can be manipulated over time, where as the standard string class is immutable.  Since it is not related to the standard string class it does not have some of the same methods to compare or convert the string.  The documentation states this is to help prevent any accidental/malicious exposure and suggests using the [Marshal](https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.marshal(v=vs.110).aspx) class.

To use it:

{% highlight csharp %}
//  Could take input from console or another api
var secureString = new SecureString();

secureString.AppendChar('s');
secureString.AppendChar('e');
secureString.AppendChar('c');
secureString.AppendChar('u');
secureString.AppendChar('r');
secureString.AppendChar('e');

IntPtr secureStringPointer = IntPtr.Zero;
try
{
  // copy to unmanaged memory and decrypt
  secureStringPointer = Marshal.SecureStringToGlobalAllocUnicode(secureString);

  // use the pointer with a function like Win32 API logon
  // Full example at https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.marshal.securestringtoglobalallocunicode(v=vs.110).aspx
  // returnValue = LogonUser(userName, domainName, secureStringPointer, LOGON32_LOGON_INTERACTIVE, LOGON32_PROVIDER_DEFAULT, ref tokenHandle);
}
finally
{
  Marshal.ZeroFreeBSTR(secureStringPointer);
}
{% endhighlight %}

## More Information
A few .NET Api's that use the ```SecureString``` class are:

- [System.Diagnostics.Process.Start](https://msdn.microsoft.com/en-us/library/ed04yy3t(v=vs.110).aspx)
- [WPF's  System.Windows.Controls.PasswordBox](https://msdn.microsoft.com/en-us/library/system.windows.controls.passwordbox.securepassword(v=vs.110).aspx)
-  [System.Security.Cryptography.X509Certificates.X509Certificate2](https://msdn.microsoft.com/en-us/library/ms148419(v=vs.110).aspx)

You can find out more information on the [MSDN SecureString page](https://msdn.mi( System.Windows.Controls)crosoft.com/en-us/library/system.security.securestring%28v=vs.110%29.aspx). The [original announcements](http://blogs.msdn.com/b/shawnfa/archive/2006/11/01/securestring-redux.aspx) on the the .NET Security Blog  has good information on why ```SecureString``` is designed the way it is.
I did use it in some the [samples](https://github.com/jsturtevant/DotNetTour) but you will find that I don't have to many real world examples. Taking password input from a WPF form and encrypting a X509 Certificate seems to be one of the few useful scenarios available today.   If you have any good examples on how to use this class let me know.
