---
layout: post
title:  "MultiValue Dictionary"
date:   2014-09-04
categories: csharp
---

The .NET team had been working on modularizing and shipping code faster than it has in the past.  One mechanism it has been using to release code earlier and faster is [NuGet](https://www.nuget.org/).  Using Nuget they are able to ship us pre-release software that we can experiment with. Enter the *MultiValueDictionary*.

## A Dictionary of Lists
Many of applications have the need for keeping track of a key and lists associated with them.  An example I came across recently was storing user roles in memory.  A single user could have multiple roles and so a standard dictionary would not work.  Traditionally, this meant having a dictionary of lists like so:

```
var roles = new Dictionary<string, List<Role>>();
```

Accessing the roles and adding a new role was cumbersome.  The code to access the list and add new roles could be abstracted away into a class that wrapped this data structure and provided simpler methods.  Until now that had to be done by the developer.  

## MultiValue Dictionary
Using NuGet, Microsoft has shipped a [pre-lease version](https://www.nuget.org/packages/Microsoft.Experimental.Collections/) of the the *MultiValueDictionary* which provides this functionality as part of the framework:

```
var roles = new MultiValueDictionary<string, Role>();
roles.Add("id", new Role());
```

You can find out more about the use of this data structure from the Microsoft blog post introducing the [MultiValueDictionary](http://blogs.msdn.com/b/dotnet/archive/2014/06/20/would-you-like-a-multidictionary.aspx). This is in pre-release but the functionality is exciting.  Hope you get to check it out.