---
layout: post
title: Helpful Cluster API commands for Devs
date: "2021-09-30"
categories:
  - kuberentes
  - cluster-api
---

If you are doing development or working with Kubernetes [cluster api](https://cluster-api.sigs.k8s.io/) these are some helpful tips.

## Handy tools

- [clusterctl](https://github.com/kubernetes-sigs/cluster-api/releases/download/v0.3.14/clusterctl-linux-amd64) - helps with cluster creation but more importantly cluster debugging
- [kubie](https://github.com/sbstp/kubie) - allows you to connect to multiple clusters at same as well as switch namespaces and clusters quickly. Helpful becuase it lets you be connected to both the management cluster and the workload cluster.

## Getting a kind management cluster kubeconfig 

```
kind get kubeconfig --name capz-e2e > kubeconfig.e2e
```

## Getting a Workload Cluster kubeconfig

When connected to management cluster

```
kubectl get clusters                        
NAME               PHASE
capz-conf-q6vvi1   Provisioned
```

Now download the clusters kubeconfig:

```
clusterctl get kubeconfig capz-conf-q6vvi1 > kubeconfig.e2e.conformance.windows
```

## Viewing the state of the cluster

https://cluster-api.sigs.k8s.io/clusterctl/commands/describe-cluster.html

```
clusterctl describe cluster capz-conf-q6vvi1
NAME                                                                 READY  SEVERITY  REASON                   SINCE  MESSAGE                                                                               
/capz-conf-q6vvi1                                                    True                                      2m53s                                                                                        
├─ClusterInfrastructure - AzureCluster/capz-conf-q6vvi1              True                                      6m16s                                                                                        
├─ControlPlane - KubeadmControlPlane/capz-conf-q6vvi1-control-plane  True                                      2m53s                                                                                        
│ └─Machine/capz-conf-q6vvi1-control-plane-845sj                     True                                      2m55s                                                                                        
└─Workers                                                                                                                                                                                                   
  ├─MachineDeployment/capz-conf-q6vvi1-md-0                                                                                                                                                                 
  └─MachineDeployment/capz-conf-q6vvi1-md-win                                                                                                                                                               
    └─2 Machines...                                                  False  Info      WaitingForBootstrapData  3m15s  See capz-conf-q6vvi1-md-win-645fbb7c79-jhjjt, capz-conf-q6vvi1-md-win-645fbb7c79-vf44j
```

## Using kubie

Use the kubeconfigs above to load a cluster:

```
kubie ctx -f kubeconfig.e2e
```

or if the cluster is in your default kubeconfig:

```
kubie ctx kind-capz
```

## ssh'ing to capz machines

ssh'ing made easy in [cluster api for azure (capz)](https://capz.sigs.k8s.io/) VM's and VMSS: 

https://github.com/kubernetes-sigs/cluster-api-provider-azure/tree/main/hack/debugging#capz-ssh

Find the machine: 

```
kubectl get azuremachine
NAME                                 READY   STATE
capz-cluster-0-control-plane-5b5fc   true    Succeeded
capz-cluster-0-md-0-fljwt            true    Succeeded
capz-cluster-0-md-0-wbx2r            true    Succeeded
```

Now ssh to it:

```
kubectl capz ssh -am capz-cluster-0-md-0-wbx2r            
```

## Deleting all of your dev clusters

Don't do this in prod! :-) 

```
kubectl delete cluster -A --all --wait=false
```

## Add vim to Windows nodes

Sometimes you need to edit the files while developing: 

```
iwr -useb get.scoop.sh | iex
scoop install vim
```

## What else?
What are your favorites? What am I missing?
