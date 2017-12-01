---
layout: post
title: Using Helm to Deploy Azure Functions on Kubernetes
date: "2017-11-30"
categories:
  - azure-functions
  - kubernetes
  - azure
---

In the previous post on running [Azure Functions on Kubernetes](posts/Running-the-Azure-Functions-runtime-in-kubernetes/) we deployed everything using manual commands.  To improve upon the solution, I created a [Helm Chart](https://docs.helm.sh/developing_charts/#charts) that enables you to deploy the Function Application.  There are a few advantages to this:

- Simplified upgrades
- Simplify the deployment of multiple Functions
- Simplify the CI/CD of multiple Function App

Checkout the chart at https://github.com/jsturtevant/azure-functions-kubernetes-chart. There are a few improvements that need to be made, such as setting up with Secrets for a private repository to pull the image.  PR's are welcome :-).

## Usage
To use, `clone` the [repository](https://github.com/jsturtevant/azure-functions-kubernetes-chart) and `cd` into the folder and run:

```bash
helm install --set functionApp.name=sampleapp \
             --set resources.requests.cpu=200m \
             --set image.repository=vyta/functions \
             --set scale.maxReplicas=10 \
             --set scale.minReplicas=1 \
             --set scale.cpuUtilizationPercentage=50 \
             ./az-func-k8
```