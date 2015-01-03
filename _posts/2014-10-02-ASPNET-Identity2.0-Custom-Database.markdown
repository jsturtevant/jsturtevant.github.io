---
layout: post
title:  "ASP.NET Identity 2.1 Custom Database"
date:   2014-10-02
categories: ASP.NET
---
<p class="message">This is a two post series.  You can read the second post at <a href="/posts/ASPNET-Identity-Custom-Database-and-OWIN">ASPNET Identity Custom Database and OWIN.</a>
</p>


I have to admit when I first took a deeper look at the [ASP.NET's Identity model](http://www.asp.net/identity) in version 2.1 I was a bit overwhelmed.  I was looking for a way to authenticate a user against an existing database using Forms authentication to create an authentication cookie after verifying the user's credentials.  Looking at the template ASP.NET Individual User Accounts I found that the ```SignInManager``` was responsible for creating the authentication cookie and so I started there. 

I found that the ```SignInManager``` constructor required a ```UserManager```  which in turned required an implementation of ```IUserStore```. And so I went in search of a way to implement my own UserStore by look at the Entity Framework  [User Store](http://msdn.microsoft.com/en-us/library/dn613259(v=vs.108).aspx).  When I saw the number of interfaces Entity Frame works UserStore implemented I froze.  Would I know what to implement and how?

## IUser\<TKey\>
So I took a few deep breaths and when went back to the ```SigninManager```.  I would start there and see what was need to implement it. Using the existing ```SigninManager``` as a reference, I found I need to have a ```IUser<TKey>```. This is easy enough:

{% highlight csharp %}
public class CustomUser : IUser<string>
{
    public string Id { get; set; }

    public string UserName { get; set; }
}
{% endhighlight %}

##  UserManager\<TUser\>
Next the ```SigninManager``` constructor takes a ```Usermanager```.  Again this was a fairly simple implementation:

{% highlight csharp %}
public class CustomUserManager : UserManager<CustomUser>
{
    public CustomUserManager(IUserStore<CustomUser> store)
        : base(store)
    {
    }
}
{% endhighlight %}

## SigninManager\<TUser, TKey\>
Now we have all the piece to be able to put together the SigninManager:

{% highlight csharp %}
public class CustomSignInManager : SignInManager<CustomUser, string>
{
    public CustomSignInManager(CustomUserManager userManager, IAuthenticationManager authenticationManager)
        : base(userManager, authenticationManager)
    {
    }
}
{% endhighlight %}

##  IUserStore\<TUser\>
The astute reader will have notice that we missed the implementation of the ```IUserStore<TUser>``` for the ```CustomerUserManager```.  This again is a very quick implementation of a few CRUD operations (I have left some of the implementation blank and the others are not production ready):

{% highlight csharp %}
public class CustomUserStore : IUserStore<CustomUser>
{
    private CustomDbContext database;

    public CustomUserStore()
    {
        this.database = new CustomDbContext();
    }

    public void Dispose()
    {
        this.database.Dispose();
    }

    public Task CreateAsync(CustomUser user)
    {
        // TODO 
        throw new NotImplementedException();
    }

    public Task UpdateAsync(CustomUser user)
    {
        // TODO 
        throw new NotImplementedException();
    }

    public Task DeleteAsync(CustomUser user)
    {
		// TODO 
        throw new NotImplementedException();
    }

    public async Task<CustomUser> FindByIdAsync(string userId)
    {
        CustomUser user = await this.database.CustomUsers.Where(c => c.UserId == userId).FirstOrDefaultAsync();
        return user;
    }

    public async Task<CustomUser> FindByNameAsync(string userName)
    {
       CustomUser user = await this.database.CustomUsers.Where(c => c.UserName == userName).FirstOrDefaultAsync();
        return user;
    }
}
{% endhighlight %}

## Next Steps
We have now implemented all the parts required to create a Custom ```SigninManager```.  The initial feeling of being overwhelmed with the number of interfaces to implement has gone away.  It was actually quite simple to create all the pieces to create our own ```SigninManager```.  Stay tuned for the next blog post where I show how to hook it into the OWIN Middleware so we can use it to sign our user in and create our Authentication Cookie.
