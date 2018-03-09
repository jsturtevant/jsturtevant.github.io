---
layout: post
title: Windows Containers Cheat Sheet
date: "2018-03-07"
categories:
  - windows containers
  - docker
---

I have been using windows containers a lot in the last month and the other day I was asked how to do something.  I don't remember anything, I use a combination of GitHub, OneNote, and Bingle (Bing/Google) for that, so of course I started looking for the various examples in various GitHub repo's that I've used and written.  Turns out this is not very efficient.  

Instead, I am going to create this living document as a Windows Container Cheat Sheet ([this blog is on GitHub so you can submit a PR](https://github.com/jsturtevant/jsturtevant.github.io/edit/master/_posts/2018-03-07-Windows-Containers-Cheat-Sheet.md) if I missed anything you think is useful).  It will serve as a quick reference for myself but hopefully can help beginners get a lay of the land.

This first section has [general links about Windows Containers](#general-info), jump to the [dev resources](#development-resources-and-tips) if your already familiar.

## General Info

### Where to find 
The first place you should know about is the [Official Windows Container Docs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/).  

### Windows Container Flavors
There are two flavors of Windows Containers: 

- [Windows ServerCore](https://hub.docker.com/r/microsoft/windowsservercore/) - Use for legacy Applications (Lift and Shift). Includes full .NET framework, and can run IIS.  Large Container size (10+ GB's)
- [Windows Nano Server](https://hub.docker.com/r/microsoft/nanoserver/) - Use for Cloud-First Applications. Small version (100's of MB)

### Windows Container Versions
To increase the speed of improvements and releases the team had to make breaking changes between versions.  This means you have to match the host machine version to the container version.  If you upgrade your host machine you can run older version of containers in Hyper-v mode.

Read more about [Windows Container version compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility).

The are [two release channels](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview):

- Long Term Support Channel (ltsc) - supported for 5 years from release
- Semi-Annual Channel (sac) - supported for 18 months from release

The current version's are:

- Windows Server 2016 (ltsc)
- Windows Server 1709 (sac)

> Note: if you are running nanoserver it only has the Semi-Annual Channel Release (sac)

When using the containers it is always a good idea to explicitly tag the images to a version an example below (choose the latest from tags on [servercore](https://hub.docker.com/r/microsoft/windowsservercore/tags/) and [nanoserver](https://hub.docker.com/r/microsoft/nanoserver/tags/)):

```dockerfile
# for an image with a specific patch in 1709
FROM microsoft/nanoserver:1709_KB4043961

# for an image with a specific path in 2016
FROM microsoft/nanoserver:10.0.14393.1770
```

## Development Resources and Tips
There are also sorts of tricks and tips that you can use.  You should checkout [Stephan's](https://github.com/StefanScherer/dockerfiles-windows) and [Elton's](https://github.com/sixeyed/dockerfiles-windows) GitHub Repo's for great examples on how to containerize almost anything.

### Download files
There are [several ways to download](https://blog.jourdant.me/post/3-ways-to-download-files-with-powershell) files. Soon you will be able to use [curl](https://blogs.technet.microsoft.com/virtualization/2017/12/19/tar-and-curl-come-to-windows/).

```dockerfile
RUN Invoke-WebRequest -UseBasicParsing  -Uri $url -OutFile 'outfile.zip'; 
```

### Extract Files
Soon you will be able to use [tar](https://blogs.technet.microsoft.com/virtualization/2017/12/19/tar-and-curl-come-to-windows/).

```dockerfile
RUN Expand-Archive outfile.zip -DestinationPath C:\temp\;
```

### Run Executable (installer)

```dockerfile
RUN Start-Process you-executable.exe -ArgumentList '--paramter', 'value' -NoNewWindow -Wait;
```

### Set Environment variable

```dockerfile
RUN setx /M ENV_VARIABLE value; 
```

### User Chocolately as a package Provider in Powershell

```dockerfile
RUN Install-PackageProvider -Name chocolatey -RequiredVersion 2.8.5.130 -Force; \
    Install-Package -Name webdeploy -RequiredVersion 3.6.0 -Force;
```

### Use escape character to chain commands

```dockerfile
# escape=`
FROM microsoft/windowsservercore

RUN Write-Host 'Line 1.'; `
    Write-Host 'Line 2';
```

### Debug .NET Framework app in Container
Instructions at [https://www.richard-banks.org/2017/02/debug-net-in-windows-container.html](https://www.richard-banks.org/2017/02/debug-net-in-windows-container.html). 

### Enable Web Auth in IIS
This also demonstrates how to set web.config files in asp.net.

```dockerfile
FROM microsoft/aspnet:4.7.1-windowsservercore-1709

RUN powershell.exe Add-WindowsFeature Web-Windows-Auth
RUN powershell.exe -NoProfile -Command `
  Set-WebConfigurationProperty -filter /system.WebServer/security/authentication/AnonymousAuthentication -name enabled -value false -PSPath IIS:\ ; `
  Set-WebConfigurationProperty -filter /system.webServer/security/authentication/windowsAuthentication -name enabled -value true -PSPath IIS:\ 
```

### Powershell Core in 1709
The [nanoserver with Powershell Core](https://hub.docker.com/r/microsoft/powershell/) installed:

```dockerfile
FROM microsoft/powershell:6.0.1-nanoserver-1709
```

### Use MultiStage Builds
Given nanoserver doesn't have full dotnet framework and 1709 doesn't ship with powershell but you can leverage multistage builds to do fancier things (like use powershell) then ship a smaller container:

```dockerfile
FROM microsoft/windowsservercore:1709 as builder

RUN Write-Host 'Use Powershell to download and install';

## ship a smaller container
FROM microsoft/nanoserver:1709

COPY --from=builder /app /app

CMD ["yourapp.exe"]
```

### General Trouble shooting
There are some great tips on how to find logs and debug issues you might run into at [https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting](https://docs.microsoft.com/en-us/virtualization/windowscontainers/troubleshooting).
