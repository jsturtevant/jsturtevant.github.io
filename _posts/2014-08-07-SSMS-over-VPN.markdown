---
layout: post
title:  "SQL Server Management Studio (SSMS) over VPN"
date:   2014-08-6
categories: SSMS
---

As with most days, today I learned something new.  This is a great tip that was passed onto me by one of my clients.  Currently, as a contractor, I work for many different companies.  Almost always, I have to do some sort of database work involving Microsoft SQL Server over a VPN.

Generally, I am given access to SQL Server using Windows Authentication based on my client's domain.  Until today I always had to log-in into one of their machines using Remote Desktop in order for SQL Server Management Studio (SSMS) to recognize my identity on the client's domain. 

Logging into remote machines is always a pain: getting/setting permission for remote access to the machine, restricted to a single monitor (RDP doesn't work well on multiple monitors), *Alt-Tab* doesn't work like it should... the little annoyances add up. 

But no longer... This simple trick will allow you to use your local instance of SSMS:

## Solution
1. Find your SQL Server Management Studio EXE.  Usually located somewhere like:
 ```C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe ``` 

2. Create a shortcut to it on the desktop.
3. Right click and select properties.
4. Add the following **before** the Target information: 
```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly```

5. Add ```-nosplash``` after Ssms.exe after the Target information
6. Your Target should look like: 
```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly "C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe -nosplash"```

7. Click OK to save your changes.
8. Make sure you are logged into the VPN for the domain your accessing and launch the shortcut.
9. You will be prompted for a password at a command prompt.  
10. Enter you password and hit enter (use the client's domain password).
11. SSMS will launch.  **Note:** Windows authentication might still show as your computers main domain but if you try to connect to a SQL Server instance in your client's domain you should connect!

The setup is a few steps but once you are up and running it is really simple to start the correct instance.

## One Step Further
I love working on the command line, especially since I started using [Cmder](http://bliker.github.io/cmder/).  So naturally, I turned the shortcut outlined above into a simple batch script that 
I dropped into Cmder's bin folder. This gave me access to call it from the command line.

Create a Batch script named ```remoteSSMS.bat``` in the Cmder bin folder and copy/paste the same information into the script as you did for the Shorcut Target information above (changing location of ssms.exe and your domain/username):

```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly "C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe -nosplash"```

Now open up Cmder and type ```remoteSSMS``` and you will be prompted for a password and SSMS will open!

Hope you find the trick as helpful as I did.