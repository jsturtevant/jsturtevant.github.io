---
layout: post
title: Starting the Service Fabric Local Cluster Manager
date: "2016-06-01"
categories:
  - service fabric
---

I have had to start the Service Fabric Cluster Manager a few times without starting Visual Studio and deploying the application for talks or demos.  Since the Cluster Manager is a separate process there is no need to run Visual Studio, simply:

1. Open the Windows Run Dialog (```Windows Key + R```)
2. Type ```ServiceFabricLocalClusterManager.exe``` and hit OK

Now your local Service Fabric Cluster is running and you can navigate to any endpoints you have exposed.