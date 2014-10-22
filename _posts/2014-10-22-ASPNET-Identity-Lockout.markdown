---
layout: post
title:  "ASPNET-Identity-Lockout"
date:   2014-10-21
categories: ASP.NET
---

This is another case for why [naming things is hard](http://martinfowler.com/bliki/TwoHardThings.html).  It might seem obvious to some but I definitely didn't get this one right away, and I wasn't the [only one](https://aspnetidentity.codeplex.com/workitem/2178).  

The ```LockoutEnabled``` flag in the ASP.NET Identity 2.0 model means that the user *can be locked out*, **not** that the user *is* locked out.  

For a user to be locked out the ```LockoutEnabled``` must be true **and** ```LockoutEndDateUtc``` must be greater than the current date.  To enable Locking out globally you must set ```UserLockoutEnabledByDefault``` to ```true``` on the ```UserManager```:

{% highlight csharp %}
public static ApplicationUserManager Create(IdentityFactoryOptions<ApplicationUserManager> options, IOwinContext context)
{
    var manager = new ApplicationUserManager(new UserStore<ApplicationUser>(context.Get<ApplicationDbContext>()));
    
	// Enable Lock outs
    manager.UserLockoutEnabledByDefault = true;
    manager.MaxFailedAccessAttemptsBeforeLockout = 5;

    // if you want to lock out indefinitely 200 years should be enough
    manager.DefaultAccountLockoutTimeSpan = TimeSpan.FromDays(365*200);

	....
}
{% endhighlight %}

This one was easy to figure out. Because the Microsoft ASP.NET Identity Team is [tracking discussions publicly](https://aspnetidentity.codeplex.com/SourceControl/latest#Readme.markdown) I was able find a [work item](https://aspnetidentity.codeplex.com/workitem/2178) with a google/bing search.  At least they didn't name it [LockoutManager](http://blog.codinghorror.com/i-shall-call-it-somethingmanager/).

