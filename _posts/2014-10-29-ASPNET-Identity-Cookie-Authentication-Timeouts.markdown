---
layout: post
title:  "ASP.NET-Identity-Cookie-Authentication-Timeouts"
date:   2014-10-21
categories: ASP.NET
---

If you are using cookie authentication in ASP.NET Identity 2.1, there are two timeout settings that look similar upon first glance:  ```ValidateInterval``` and ```ExpireTimespan```:

{% highlight csharp %}
 app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    AuthenticationType = DefaultAuthenticationTypes.ApplicationCookie,
    LoginPath = new PathString("/Account/Login"),
    Provider = new CookieAuthenticationProvider
    {
        OnValidateIdentity = SecurityStampValidator.OnValidateIdentity<ApplicationUserManager, ApplicationUser>(
            validateInterval: TimeSpan.FromMinutes(15),
            regenerateIdentity: (manager, user) => user.GenerateUserIdentityAsync(manager)),
    },
    SlidingExpiration = false,
    ExpireTimeSpan = TimeSpan.FromMinutes(30)
});         
{% endhighlight %}

## ExpireTimeSpan
```CookieAuthenticationOptions.ExpireTimespan```  is the option that allows you to set how long the issued cookie is valid for.  In the example above the cookie is valid for 30 minutes from the time of creation.  Once those 30 minutes are up the user will have to sign back in becuase the ```SlidingExpiration``` is set to ```false```.  

If ```SlidingExpiration``` is set to ```true``` then the cookie would be [re-issued](http://msdn.microsoft.com/en-us/library/microsoft.owin.security.cookies.cookieauthenticationoptions.slidingexpiration(v=vs.113).aspx) on any request half way through the ```ExpireTimeSpan```.  In the example above, if the user logged in and then made a second request 16 minutes later the cookie would be re-issued for another 30 minutes.  If the user logged in and then made a second request 31 minutes later then the user would be prompted to log in.

## SecurityStampValidator Validation Interval
The ```validateInterval``` of the ```SecurityStampValidator.OnValidateIdentity``` function  checks the security stamp field to insure validity after the given internval.  This is not the same as checking expiration of the cookie, although it can cause the same result of being logged out.  

The Security Stamp is created anytime a password is created/changed or an external login is added/removed.  If a user changes their password then the SecurityStamp will be updated.  This results in any cookie that have been issued previous to the password to become invalid the next time the ```validateInterval``` occurs.  

For a concrete example using the above settings (this is a unlikely example but gets the point across): 

1. User signs in at location A. 
2. Same user changes work stations and signs in 10 minutes later at location B 
3. Same user changes their password at location B at the 12 minute mark
4. The Same user goes back the the work station at location A and issues a request at the 20 minute mark.  
5. Since the user issued a request after the ```validateInterval``` at location A they will be signed-out and prompted for their credentials again.

This is different from the the cookie timing out because the 30 minute Expire Time out was never reached but yet the user is logged out because the ```validateInterval``` of the SecurityStampValidator.OnValidateIdentity was set to 15 minutes.


## The differences
The difference is subtle at first glance but provides some great benefits, such as [Sign-out Everywhere](https://aspnet.codeplex.com/SourceControl/latest#Samples/Identity/SingleSignOutSample/readme.txt).  But it can be confusing since the default ASP.NET Identity template only has ```validateInterval``` leaving the ```ExpireTimespan``` to the default of 14 days.   Without some digging a developer new to the ASP.NET Identity library might not immediately recognize that the ```validateInterval``` is not the same as expiring cookies on a given time fame.




