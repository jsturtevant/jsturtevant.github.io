---
layout: post
title:  "ASPNET Identity Custom Database and OWIN"
date:   2014-10-09
categories: ASP.NET
---

In the last [post](), we covered how to create a custom ```SigninManager```.  At first look there was a lot of work to be down but after diving in we found that there were only a few simple classes we had to set up.  

In this post I will demonstrate how to use [OWIN]() to load our ```SigninManager``` for each request.  By the end of the post we will have a ```SigninManager``` that we can use to create an authentication cookie and sign a user into our system.

##  OWIN Middleware
The latest version of ASP.NET is setup to use the OWIN middleware.  If we take a look the ```Startup.Auth.cs``` file in the ```App_Start``` folder we will find an example of how the out-of-the-box ```SigninManager``` is configured:

{% highlight csharp %}
public void ConfigureAuth(IAppBuilder app)
{
    app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
    app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

	//left other configuration out for brevity
}
{% endhighlight %}

The ```app.CreatePerOwinContext<T>();``` creates one instance of the given type per request.  This way we only have ApplicationUserManager and one ApplicationSignInManager around for each request.  To configure our  CustomUserManager and CustomSignInManager we need to create factory methods and add them to the ```Startup.Auth.cs```.  The ```app.CreatePerOwinContext<T>();``` can take two different types of methods (We will see each type of call back method when we create them):

1. ```Func<IdentityFactoryOptions<T>, IOwinContext, T>``` 
2. ```Func<T> createCallback```

It is important to note the order in which the ```app.CreatePerOwinContext<T>();``` is called.  Since a ```SigninManager``` requires a ```UserManger``` the call to create the ```SigninManager``` on the OWIN context must come after the call to create the ```UserManager```

## Creating a Custom SigninManager

{% highlight csharp %}
public static CustomUserManager Create()
{
    var manager = new CustomUserManager(new CustomUserStore());

    return manager;
}
{% endhighlight %}

{% highlight csharp %}
public static CustomSignInManager Create(IdentityFactoryOptions<CustomSignInManager> options, IOwinContext context)
{
    return new CustomSignInManager(context.GetUserManager<CustomUserManager>(), context.Authentication);
}
{% endhighlight %}