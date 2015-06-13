---
layout: post
title:  "ASP.NET Identity Custom Database and OWIN"
date:   2014-10-09
categories:
  - asp.net
  - csharp
---

<p class="message">This is a two post series.  You may like to read the first post <a href="/posts/ASPNET-Identity2.0-Custom-Database">ASPNET Identity and Custom Database.</a>
</p>

In the last [post](/posts/ASPNET-Identity2.0-Custom-Database/), we covered how to create a custom ```SigninManager```.  At first glance there was a lot of work to be down but after diving in we found that there were only a few simple classes we had to set up.  

In this post, I will demonstrate how to use [OWIN](http://www.asp.net/aspnet/overview/owin-and-katana) to load our ```SigninManager``` for each request.  By the end of the post we will have a ```SigninManager``` that we can use to create an authentication cookie and sign a user into our system.

##  OWIN Middleware
The latest version of ASP.NET is setup to use the OWIN middleware.  If we take a look the ```Startup.Auth.cs``` file in the ```App_Start``` folder we will find an example of the out-of-the-box ```SigninManager``` configuration:

{% highlight csharp %}
public void ConfigureAuth(IAppBuilder app)
{
    app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
    app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

	//left other configuration out for brevity
}
{% endhighlight %}

The ```app.CreatePerOwinContext<T>()``` creates one instance of the given type per request.  This way we only have ApplicationUserManager and one ApplicationSignInManager around for each request.  To configure our  CustomUserManager and CustomSignInManager we need to create factory methods and add them to the ```Startup.Auth.cs```.  

It is important to note the order in which the ```app.CreatePerOwinContext<T>()``` is called in the configuration.  Since a ```SigninManager``` requires a ```UserManger``` the call to create the ```SigninManager``` on the OWIN context must come ***after*** the call to create the ```UserManager```

The ```app.CreatePerOwinContext<T>()``` can take two different types of methods (We will see each type of callback method when we create them):

1. ```Func<T> createCallback```
2. ```Func<IdentityFactoryOptions<T>, IOwinContext, T>```

## UserManager Factory Method
The UserManager method is the first callback method type; a simple static method that returns a new ```CustomUserManager```.  It would be possible to configure a method that also returns the CustomUserDataContext and pass it to the store. To keep it simple and show both types of methods ```app.CreatePerOwinContext<T>()``` takes I have left it out.

{% highlight csharp %}
public static CustomUserManager Create()
{
    var manager = new CustomUserManager(new CustomUserStore());
    return manager;
}
{% endhighlight %}

## SignInManager Factory Method
The ```SigninManager``` has a the second callback type but is again not too complex:

{% highlight csharp %}
public static CustomSignInManager Create(IdentityFactoryOptions<CustomSignInManager> options, IOwinContext context)
{
    return new CustomSignInManager(context.GetUserManager<CustomUserManager>(), context.Authentication);
}
{% endhighlight %}

##  OWIN Configuration
The configuration is very similar to the out-of-the-box configuration.  In the ```Startup.Auth.cs``` file we add our custom managers:

{% highlight csharp %}
public void ConfigureAuth(IAppBuilder app)
{
    app.CreatePerOwinContext<ApplicationUserManager>(ApplicationUserManager.Create);
    app.CreatePerOwinContext<ApplicationSignInManager>(ApplicationSignInManager.Create);

	// Add our custom managers
	app.CreatePerOwinContext<CustomUserManager>(CustomUserManager.Create);
    app.CreatePerOwinContext<CustomSignInManager>(CustomSignInManager.Create);
}
{% endhighlight %}

## Use in a Controller
And to use it in a Controller add the following property to the Controller class which uses the HttpContext to get our single instance per request.  The private ```set``` allows you to create a Controller Constructor where you can pass a mock instance for testing.

{% highlight csharp %}
private CustomSignInManager _signInManager;
public CustomSignInManager CustomSignInManager
{
    get
    {
        return _signInManager ?? HttpContext.GetOwinContext().Get<CustomSignInManager>();
    }
    private set { _signInManager = value; }
}
{% endhighlight %}

Now in your your Login Action use it:

{% highlight csharp %}
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<ActionResult> Login(LoginViewModel model, string returnUrl)
{
   // do model validations, password compare and  other checks then sign them in

	//creates and signs a cookie.
	 CustomSignInManager.SignIn(customUser, isPersistent: true, rememberBrowser: false);
}
{% endhighlight %}

Hope that helps clear up some of the steps that are required to create an authentication cookie against an existing database.  
