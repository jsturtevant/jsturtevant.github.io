---
layout: post
title: 'How to view, add, edit and remove files in Azure Web App using the Kudu service Dashboard'
date: '2016-01-07'
categories:
  - azure
  - webapps
---

While debugging your Azure Web App deployments it is sometimes useful to view the files that are deployed to the service.  Several situations come to mind.  For example, trying to debug your database connection strings or possibly determining [which files actually got deployed and where they sit in the file system](http://stackoverflow.com/questions/24497774/configure-python-3-4-and-django-on-windows-azure).  Or maybe you need to check if all your dependencies have been installed.   Maybe a [firewall blocks your FTP access](https://wiki.filezilla-project.org/Network_Configuration).

But how do you do view the files if you are using a [Platform as a Service (PaaS)](https://en.wikipedia.org/wiki/Platform_as_a_service) solution such as [Web Apps](https://azure.microsoft.com/en-us/services/app-service/web/)? PaaS doesn't [set up a virtual machine](setting-up-postgresql-in-azure-vm/) that you can remote into in the traditional sense and instead abstracts all that complexity away for you.  Enter the [Kudu](https://github.com/projectkudu/kudu/wiki) Service Dashboard.

## What is Kudu?
[Kudu](https://github.com/projectkudu/kudu/wiki) is a set of tools and extensibility points for App Service applications.  Anytime you set up a [git deployment](https://azure.microsoft.com/en-us/documentation/articles/web-sites-publish-source-control/) in Azure Web Sites, Kudu is running and managing the deployment.  Kudu also allows you to see environment variables, see processes running on the machine, use a cmd console and much more.  It also provides a way for you to create [extensions for Azure Websites](https://github.com/projectkudu/kudu/wiki/Azure-Site-Extensions) that give you powerful capabilities like [Image Optimization](https://github.com/ligershark/AzureJobs) or adding [Go support](https://github.com/wadewegner/azure-go-lang-site-extension).   

 Not only can it do [all that and more](https://github.com/projectkudu/kudu/wiki#features) but it can also [run outside of Azure on your on server](https://github.com/projectkudu/kudu/wiki/Deploying-to-a-server).  You can even [fork the source code](https://github.com/projectkudu/kudu) and [tell Azure to use your own version of Kudu](https://github.com/projectkudu/kudu/wiki/Deploy-locally-built-private-kudu-to-azure).
 
 How do you start using it?

## How to view Kudu service dashboard
There are [two ways to access Kudu](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service):

1. Simply modify your website URL and by adding `scm` to it. If you site is `http://mysite.azurewebsites.net/`, then the root URL of the Kudu service is `https://mysite.scm.azurewebsites.net/`. Note the added `scm` token.

2. Using the Azure Portal.  First Navigate to your Web App, Select `Tools` -> `Kudu` -> `Go`:

![launch Kudu from azure portal]({{ site.url }}/assets/Azure-Kudu-Service-how-to.png)

## How to View, Add, Edit, and Remove files in Azure Web App using Kudu
Finally this post was about how you actually view, edit, add, and remove files from the Web App.  Once you have your Kudu service Dashboard open you will see some basic information and links for more complex tasks:

![Kudu home page]({{ site.url }}/assets/Azure-Kudu-Service-home.png)

### View Current Files
View current files in your application by Clicking on `Debug Console` -> `CMD`:

![Kudu view files]({{ site.url }}/assets/azure-Kudu-service-view-files.png)

Once you are viewing the folder structure you can get to your application home directory by clicking the `site` folder:

![Kudu site folder]({{ site.url }}/assets/azure-Kudu-service-site-folder.png)

And then the double clicking `wwwroot` folder:

![Kudu site folder]({{ site.url }}/assets/azure-Kudu-service-wwwroot-folder.png)

This is where all your files live.  You can even [customize this location](https://github.com/projectkudu/kudu/wiki/Customizing-deployments).  In this case, only the default starter HTML page is available but in the case of a larger application there would be many files.

![Kudu your files live here]({{ site.url }}/assets/azure-Kudu-service-files-live-here.png)

### Edit Files
To edit a file click the `pencil` icon:

![launch Kudu from azure portal]({{ site.url }}/assets/azure-Kudu-service-files-edit.png)

### Add Files
To add a file you can drag it from your file system to the folder.  It is important to drag it directly onto the folder viewer.

![launch Kudu from azure portal]({{ site.url }}/assets/azure-Kudu-service-files-add.png)

### Remove Files
Toe remove a file click the `minus` icon:

![launch Kudu from azure portal]({{ site.url }}/assets/azure-Kudu-service-files-remove.png)

## Conclusion
It is really simple to use the [Kudu](https://github.com/projectkudu/kudu/wiki) Service to view, edit, add, and remove files from your Web App.  But as you can see there are a lot of capabilities that [Kudu](https://github.com/projectkudu/kudu/wiki) brings to Azure Web apps. Another really useful ability that Kudu gives you is the ability to [view log files](https://github.com/projectkudu/kudu/wiki/Diagnostic-Log-Stream).  

Have fun getting to know [Kudu](https://github.com/projectkudu/kudu/wiki) and all the [possibilities it brings to the table](http://blog.amitapple.com/post/56390805814/deployment-email/#.VpBq5BUrLic).