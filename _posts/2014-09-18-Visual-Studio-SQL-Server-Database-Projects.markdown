---
layout: post
title:  "Visual Studio SQL Server Database Projects"
date:   2014-09-18
categories: SSMS, SQL
---

Visual Studio is a large application with so many features that you sometimes you stumble into a new one and think *if I had know that 3 years ago.*  With all the hype over new features like [Code Lens](http://msdn.microsoft.com/en-us/library/dn269218.aspx), [Peek Definition](http://msdn.microsoft.com/en-us/library/dn160178.aspx) and [ASP.NET Integration](http://blogs.msdn.com/b/webdev/archive/2013/10/16/asp-net-features-in-new-project-templates-in-visual-studio-2013.aspx) there are some features that aren't as flashy and don't get the attention they deserve...

## SQL Server Database Projects
A co-worker of mine, Floyd, (you can find him at [@fhilton](https://twitter.com/fhilton))  recently stumbled SQL Server Database Project templates and I think he found a gold mine:

![Visual Studio 2013 SQL Server Database Project Template]({{ site.url }}/assets/vs-2013-sql-server-database-template.jpg)

A Database project get associated with a SQL Server database and allows for developers to pull in all the various assets of the database. You can also start a new database using the template and deploy right from Visual Studio.

## The Features
There are two many exciting features to cover them all but here's a list of my favorite:

- [Schema comparison](http://channel9.msdn.com/Events/Visual-Studio/Launch-2013/VS108) - compare schemas between your development database and production
- [Data comparison](http://channel9.msdn.com/Events/Visual-Studio/Launch-2013/VS108) - compare data in the tables between different databases
- Source Control - ***Yes, Source Control***
- Change Script generation
- [Unit Testing](http://channel9.msdn.com/Events/Visual-Studio/Launch-2013/QE107) - ***Yes, Unit Testing (what is this craziness?)***
- Generate Snapshots - Snapshots are a point in time version of the database that you can do a schema compare against
- Re-factoring support - Rename a table or column and have all your stored procedures update!
- Deployment
- Windows Azure SQL Database Support

And don't worry you can do all your ad hoc SQL queries right from Visual studio with out having to boot up SSMS.  You can find out more from the Channel 9 video  [Introducing SQL Server Database Projects Tooling in Visual Studio 2013](http://channel9.msdn.com/Events/Visual-Studio/Launch-2013/VS111)

I have only just started to explore the tools but can tell it is going to improve my teams database management practices.  In what ways do you see your self using these tools?  What must try features have I missed?