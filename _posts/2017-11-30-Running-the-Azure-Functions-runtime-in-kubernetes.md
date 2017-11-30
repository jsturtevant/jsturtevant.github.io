---
layout: post
title: Running the Azure Functions Runtime in Kubernetes
date: "2017-11-30"
categories:
  - azure-functions
  - kubernetes
  - azure
---

The Azure Functions team recently released the preview of [Azure Functions on Linux](https://blogs.msdn.microsoft.com/appserviceteam/2017/11/15/functions-on-linux-preview/).  A colleague [Vy Ta](https://twitter.com/vytachar) and I thought it would be fun to see if we could get Azure Functions running in Kubernetes.  Here are the steps to get it work.  To follow along you will need to have:

- [.NET Core 2.0](https://www.microsoft.com/net/download/linux)
- [Docker](https://www.docker.com/)
- [Functions Core Tools 2.0](https://github.com/Azure/azure-functions-cli)
- [Kubernetes cluster](https://docs.microsoft.com/en-us/azure/aks/)
- Docker Repository ([Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/) or [Docker Hub](https://azure.microsoft.com/en-us/services/container-registry/))  

## Create a your Function App and Docker image
The first step is to use the Functions Core Tools to create a sample function:

```bash
func init . --docker --sample
```

Next build your Docker image and push to a docker repository:

```bash
docker build -t az-functions.
docker tag az-functions <your-repo>/az-functions

docker login
docker push <your-repo>/az-functions
```

## Set up and run on Kubernetes
Next we will create a deployment and service on Kubernetes and make sure we can access it.  The commands below assume you have a Kubernetes cluster running in a Cloud.

Create a deployment on Kubernetes:

```bash
kubectl run azfunctions --image=<your-repo>/az-functions --port=80 --requests=cpu=200m
```

Create a service and wait for an IP address:

```bash
kubectl expose deployment azfunctions --port=80 --type=LoadBalancer
kubectl get svc -w
```

Once you have a ip address you should be able to open a browser and view the end point at `http://<your-ip-address>/api/httpfunction?name=james` (assuming you used the sample function).

## Autoscale your Function App
Once we have both of those running we can set up a Pod Auto Scaler and test scaling our function app.

Set up auto Scaling by:

```bash
kubectl autoscale deployment azfunctions --cpu-percent=50 --min=1 --max=10
```

> Note:  For the auto-scaler to work you need to create you deployment with the `--requests=cpu=200m` property as we did above in `kubectl run` command.  It is possible to [autoscale on other metrics as well](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).

## Test Autoscaling
We have enabled auto scaling so let give it a spin. We will test it the same way as in [Kubernetes Pod Autoscale walk through](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/).

Open a new terminal and run:

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
/ \#: 		while true; do wget -q -O- http://<your-ipaddress>/api/httpfunction?name=testingload; done 
```

This will ping your endpoint.  After a few moments you should be able to see the load increasing:

```bash
kubectl get hpa

#output
NAME          REFERENCE                TARGETS      MINPODS   MAXPODS   REPLICAS   AGE
azfunctions   Deployment/azfunctions   167% / 50%   1         10        4          5m
```

You can also see the number of pods in the deployment increase:

```bash
kubectl get deploy azfunctions

#output
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
azfunctions   4         4         4            4           10m
```

If you kill the `busybox` command we used to generate the load you will see the pods scale back down.

## Where to next
This is a great way to see how you could use the Azure Functions runtime on premise and opens a lot of possibilities.  One scenario might be that you have two teams.  One that is working On Premises and another that works in Azure but you want to share the same programming model of Azure functions across the two teams.  Another scenario is to use Kubernetes and then use [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/) for extra scaling when needed.  What do you think you might use this for?  Leave a note in the comments.

Some other interesting scenarios you can looking into are:

- Other scaling parameters using [custom metrics](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- Running the [Azure Function portal](https://github.com/Azure/azure-functions-ux) for Adminstration
- Creating a Helm Chart to manage the deployments of multiple Function Apps











