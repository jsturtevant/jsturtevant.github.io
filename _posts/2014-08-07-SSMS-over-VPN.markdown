---
layout: post
title:  "SSMS over VPN"
date:   2014-08-6
categories: SSMS
---

As with most days, today I learned something new.  This was a great tip that was passed onto me by one of my clients.  Currently as a contractor I work for many different companies.  Almost always, I have to do some sort of database work involving Microsoft SQL Server.

Generally, I am given access to SQL Server using windows authentication permissions based on my client's domain.  Until today I always had to log-in into one of their machines using Remote Desktop in order for SQL Server Management Studio (SSMS) to recognized me on the client's domain. 

Logging into remote machines is always a pain: getting/setting permission for remote access to the machine, I can only use on a single monitor (RDP does't work well on multple monitors), *Alt-Tab* doesn't work like it should... the little annoyances add up. 

But no longer... This simple trick will allow you to use your local instance of SSMS:

## Solution
1. Find your SQL Server Management Studio exe.  Usually located somewhere like: ```C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe ``` 
2. Create a shortcut to it on the desktop.
3. Right click and select properties.
4. Add the following **before** the Target information: ```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly```
5. Add ```-nosplash``` after Ssms.exe after the Target information
6. Your Target should look like: ```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly "C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe -nosplash"```
7. Click OK to save your changes.
8. Make sure you are logged into the VPN for the domain your accessing and launch the shortcut.
9. You will be prompted for the domains password at a command prompt.  
10. Enter you password and hit enter.  
11. SSMS will launch.  Windows authentication might still show as your computers main domain but if you try to connect to a SQL Server instance in your clients you should connect!

The setup is quite  a few steps but once you are up and running it is really simple to start the correct instance.

## One Step Further
I love working on the command line, especially since I started using [Cmder]().  So naturally I turned the shortcut outlined above into a simple batch script that 
I dropped into Cmder's bin folder, which give me access to call it from the command line.

Create a Batch script named ```remoteSSMS.bat``` in the Cmder bin folder and put the paste the same information into the script as we did for the Shorcut Target information above (changing location of ssms.exe and your domain/username):

```C:\Windows\System32\runas.exe /user:yourclientsdomainname\yourusername /netonly "C:\Program Files (x86)\Microsoft SQL Server\[version number]\Tools\Binn\ManagementStudio\Ssms.exe -nosplash"```

Now open up Cmder and type ```remoteSSMS``` and you will be prompted for a passowrd and SSMS will open!

Hope you find the trick as helpful as I did.