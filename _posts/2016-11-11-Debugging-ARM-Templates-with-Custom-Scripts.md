---
layout: post
title: Debugging ARM Templates with Custom Scripts
date: "2016-11-11"
categories:
  - azure
  - arm
---

A while back I was working a [one-click Azure Deploy button](https://azure.microsoft.com/en-us/blog/deploy-to-azure-button-for-azure-websites-2/) for the [Hoodie sample application](https://github.com/jsturtevant/hoodie-app-tracker).  If you have not seen [Hoodie](http://hood.ie/) before you should check the project out.  It is an opensource project with a complete backend that allows front end developers to build amazing applications and they have a [great community](https://channel9.msdn.com/Blogs/DevRadio/DR1653).

## ARM Template Debugging Issues 
I ran into an issue during development and testing of the [ARM template](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/) that powers the one click deploy.  When deploying, the template was deploying the VM successfully but the custom linux script was not completing properly.  

After several attempts someone pointed out that I made a silly mistake by including a interactive prompt in the docker command.  This caused one command in the script to time out but not actually fail the entire ARM deployment leaving me with no way to determine what went wrong or so I thought.  

## ARM Template Logs
After debugging for longer than I want to admit, I learned that the ARM templates store logs on the VM.  Knowing this would have cut my debug time down considerably.  You can find all the logs for ARM Deployments at: 

```
/var/logs/Azure
```

In my case the log I was interested in for my custom script was located at:

```
/var/log/azure/Microsoft.OSTCExtensions.CustomScriptForLinux/1.5
.2.0/extension.log
```

Hope that helps someone not have to spend hours trying different configurations over and over because a silly interactive prompt.