---
layout: post
title: Checking the Aks Acs-Engine version number for debugging
date: "2018-06-25"
categories:
  - kubernetes
  - azure
---

[Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/intro-kubernetes) uses the [acs-engine project](https://github.com/Azure/acs-engine) behind the scenes.  Acs-engine is used as place to prototype, experiment and bake features before they make it into AKS. 

I was recently working on a project where we using AKS shortly after it went General Availability (GA).  We saw strange behavior on our test cluster related to provisioning volume mounts and load balancers that we could not reproduce with newly created clusters.  We checked the version numbers on Kubernetes/code/images but we could not find anything different between the clusters.

We finally found that there was a difference between acs-engine versions of the clusters.  This happened because the customer had created the cluster before the GA date.  Recreating the cluster (and therefor getting the latest changes from acs-engine) fixed many of the inconsistencies we were seeing in the cluster with issues.   

To check an AKS cluster acs-engine version number:

```bash
# find the generated resource group name
az group list -o table
Name                       Location    Status
-------------------------  ----------  ---------
MC_vnet_kvnet_eastus       eastus      Succeeded
vnet                       eastus      Succeeded

# find a node name for that group
az vm list -g MC_vnet_kvnet_eastus -o table
Name                      ResourceGroup         Location    Zones
------------------------  --------------------  ----------  -------
aks-agentpool-8593-0  MC_vnet_kvnet_eastus  eastus
aks-agentpool-8593-1  MC_vnet_kvnet_eastus  eastus
aks-agentpool-8593-2  MC_vnet_kvnet_eastus  eastus

# list the tags to see the acs-version number
az vm show -n aks-agentpool-8593-0 -g MC_vnet_kvnet_eastus --query tags
# output
{
  "acsengineVersion": "0.16.2", 
  "creationSource": "aks-aks-agentpool-8593-0",
  "orchestrator": "Kubernetes:1.9.6",
  "poolName": "agentpool",
  "resourceNameSuffix": "8593"
}
```

Being able to compare the version numbers helped pinpoint the issue but the bigger lesson learned is to always recreate your Azure resources after a product goes GA.  There are a lot of changes, fixes and releases that happen in the weeks leading up to a product release in Azure and the best way to make sure your running the latest software is to create a resource after the GA event. 



