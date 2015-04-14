---
layout: post
title: Runaway SQL Server Service Broker Activation Script
date: "2015-04-14 17:28"
categories:
  - ssms
  - sql
---

I have been experimenting with using the [SQL Server Service Broker](https://msdn.microsoft.com/en-us/library/bb522893.aspx) in my latest project.  We had a long running SQL process that needed to be executed in the background and the Service Broker sounded like a good choice.  I found some good [articles](http://blog.sqlauthority.com/2009/09/21/sql-server-intorduction-to-service-broker-and-sample-script/) on how to set this up and had success quickly.  After some simple examples I tried the a more complex solution and set up an [activation script](https://technet.microsoft.com/en-us/library/ms171617(v=sql.105).aspx). This is when the problems started.

## Trials and Errors
I quickly found that error handling with with the Service Broker was [not straight forward](http://rusanu.com/2007/10/31/error-handling-in-service-broker-procedures/).  After implementing some of the error handling I ran the process again but this time I had a bug in my error handling which caused an infinite loop in the activation script.  This time when I ran the process it never returned...  

The SQL Server log file grew and grew.  We tried to restarting the server but when restarted the activation script just started off on an infinite loop again.  We found the process id and tried to kill the process but got the error ```Only user processes can be killed.```  We tried dropping the activation procedure, the services and the queue.  All to no avail.

## Solution
We searched to the ends of google/bing but didn't find a solution.  Finally, in a last ditch effort, we remember that we enabled the Service Broker so we tried disabling it.  It worked!  We had finally killed the process that was running rogue with the following command:

```
ALTER DATABASE dbname SET DISABLE_BROKER;
```

It was brute force and luckily we were working in the development environment.  I hope that this helps anyone with a runaway/rouge SQL Service Broker procedure.  If there is another I would love to hear about it.
