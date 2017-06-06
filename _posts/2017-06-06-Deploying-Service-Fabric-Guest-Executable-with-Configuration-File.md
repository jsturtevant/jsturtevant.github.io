---
layout: post
title: Deploying a Service Fabric Guest Executable with Configuration File
date: "2017-06-06"
categories:
  - service fabric
---

In Service Fabric, when your guest executable relies on a configuration file, the file must be put in the same folder as the executable.  An example of an application that that needs configuration is [nginx](http://nginx.org/en/docs/beginners_guide.html) or using [Traefik](https://docs.traefik.io/basics/#static-trfik-configuration) with a static configuration file.

The folder structure would look like:

```
|-- ApplicationPackageRoot
    |-- GuestService
        |-- Code
            |-- guest.exe
            |-- configuration.yml
        |-- Config
            |-- Settings.xml
        |-- Data
        |-- ServiceManifest.xml
    |-- ApplicationManifest.xml
```

You may need to pass the location of the configuration file via ```Arguments``` tag in the ```ServiceManifest.xml```.  Additionally you need to set the ```WorkingFolder``` tag to ```CodeBase``` so it is able to see the folder when it starts.  The valid options for the ```WorkingFolder``` parameter are [specified in the docs](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-deploy-existing-app#use-visual-studio-to-package-an-existing-executable).  An example configuration of the ```CodePackage``` section of the ```ServiceManifest.xml``` file is:

```xml
<CodePackage Name="Code" Version="1.0.0">
  <EntryPoint>
    <ExeHost>
      <Program>traefik_windows-amd64.exe</Program>
      <Arguments>-c traefik.toml</Arguments>
      <WorkingFolder>CodeBase</WorkingFolder>
      <ConsoleRedirection FileRetentionCount="5" FileMaxSizeInKb="2048"/> 
    </ExeHost>
  </EntryPoint>
</CodePackage>
```

Not the ```ConsoleRedirection``` tag also enables redirection of ```stdout``` and ```stderr``` which is helpful for debugging.