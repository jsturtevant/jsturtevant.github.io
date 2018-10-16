---
layout: post
title: Deploying AKS with least privileged service principle
date: "2018-10-16"
categories:
  - aks
  - azure
---

When deploying an [Azure Kubernetes Service cluster](https://docs.microsoft.com/en-us/azure/aks/) you are required to use a service principle.  This service principle is used by the [Kubernetes Azure Cloud Provider](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/) to do many different of activities in Azure such as provision IP addresses, create storage disks and more.  If do not specify a Service principle at the time of creation for the an AKS cluster a service principle is [created behind the scenes](https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal). 

In many scenarios, the resources your cluster will need to interact with will be outside the auto-generated node resource group that AKS creates.  For instance, if you are using Static IP address for some of you services, the IP addresses might live in a resource group outside the auto-generated node resource group.  Another common example is attaching to an existing vnet that is already provisioned.

## Assigning proper permissions
It would be simple to give `Contributor` rights across your whole sub or even individual resource groups and these scenarios would work but this is not a good practice.  Instead we should assign the specific, least privileged rights to a given service principle. 

In the examples of attaching IP addresses, we may need the Azure Cloud Provider to be able to attach but *not* delete IP addresses because a different team controls the IP address.  We definitely don't want the Service principle to be able to delete any other resources in the resource group.

There is a good list of the Kubernetes v1.11 permissions required [here](https://gist.github.com/noelbundick/7799d7dfe76745a4fdd31b0f8563a858). This list shows the permissions for creating a least privileged service principle (note that it might change as k8s version change so use as general guide).  Using this we can assign just enough rights to the service principle to interact with the resources outside the node group. 

## Sample Walk through
I have created a full walk through [sample of creating the service principle](https://github.com/jsturtevant/aks-examples/tree/master/least-privileged-sp) upfront and assign just enough rights to it to access the IP addresses and vnet.  Here is a highlight of the important parts:

First create a service principle with no permission:

```bash
clientsecret=$(az ad sp create-for-rbac --skip-assignment -n $spname -o json | jq -r .password)
```

Using custom Role Definitions I am able to give the service principle only read access to the IP's:

```json
{
    "Name": "AKS IP Role",
    "IsCustom": true,
    "Description": "Needed for attach of ip address to load balancer created in K8s cluster.",
    "Actions": [
        "Microsoft.Network/publicIPAddresses/join/action",
        "Microsoft.Network/publicIPAddresses/read"
    ],
    "NotActions": [],
    "DataActions": [],
    "NotDataActions": [],
    "AssignableScopes": [
        "/subscriptions/yoursub"
    ]
}
```

Using that definition we can then apply the Role Definition to the Service Principle which will give it IP read access to the resource group:

```bash
spid=$(az ad sp show --id "http://$spname" -o json | jq -r .objectId)

iprole=$(az role definition create --role-definition ./role-definitions/aks-reader.json)
az role assignment create --role "AKS IP Role" --resource-group ip-resouce-group --assignee-object-id $spid
```

Then you can deploy your cluster using the service principle:

```
az aks create \
    --resource-group aks-rg \
    --name aks-cluster \
    --service-principal $spid \
    --client-secret $clientsecret
```

This will allow the Service Principle used to access the the IP Addresses in the resource group outside the node.

> note: you will need to provide an annotation on the k8s `Service` definition (`service.beta.kubernetes.io/azure-load-balancer-resource-group`).  See the [example](https://github.com/jsturtevant/aks-examples/blob/e1ea6ebf7c3fc34e34d9ee1ce8776c1293d8a598/least-privileged-sp/k8s/deployment.yml#L24). 

Check out the [sample for a full walk through]((https://github.com/jsturtevant/aks-examples/tree/master/least-privileged-sp)).