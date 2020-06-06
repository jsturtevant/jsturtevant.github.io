---
layout: post
title: Debugging CNI and Kubelet in Kubernetes
date: "2020-06-06"
categories:
  - kubernetes
  - azure
---

I have been working on [adding IPV6 support to the Azure cluster api provider](https://github.com/kubernetes-sigs/cluster-api-provider-azure/pull/646) (CAPZ) over the last few weeks.  I ran into some trouble configuring Calico as the Container Networking Interface (CNI) for the solution.  There are some very old ([2017](https://github.com/kubernetes/kubernetes/issues/48798)) and popular ([updated 13 days](https://github.com/kubernetes/kubeadm/issues/1031#issuecomment-633419099) ago at time of writing) issues on github where folks are looking for ways to fix the issue.

This is obviously something that is hard to debug due to the error message being vague and the wide variety of CNI's available and the wide range of issues that could be causing it.  The original issues did seem to be root caused but others (me included) are still running into the issue regularly.  In my case, I was already [running the latest version of my cni](https://github.com/kubernetes/kubernetes/issues/48798#issuecomment-630397355) (plus just running the latest isn't really understanding the root cause) and I understand why [removing environment variables doesn't really help](https://github.com/kubernetes/kubernetes/issues/48798#issuecomment-321267386).  I figured I had simply configured something wrong but what was it?

## Debugging CNI issues with Kubelet
The dreaded error message from Kubelet is:

```go
kubelet: E0714 12:45:30.541001 7263 kubelet.go:2136] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message: network plugin is not ready: cni config uninitialized
```

Unfortunately this is not very descriptive.  Sometimes kubelet has other error messages in it that [give you pointers to what is wrong](https://github.com/kubernetes/kubernetes/issues/48798#issuecomment-326220343) but in my case there was nothing in the logs.  The CNI itself wasn't working but there weren't any error logs that I could find.  

After a big of digging I found the a [developer tool for executing CNI's manually](https://github.com/containernetworking/cni/tree/master/cnitool).  This allows you to attach an interface to an already created network namespace via a CNI you provide. After building the tool, it is pretty straight forward to execute it with the [cni and configuration](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni) that is installed on the kubernetes node.  

It assumes the cni config is installed to `/etc/cni/net.d` and `/opt/cni/bin` where cni's are typically installed if you use [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):

```
# create a test network namespace
sudo ip netns add testingns

# run with default config location
sudo CNI_PATH=/opt/cni/bin cnitool add uniquename /var/run/netns/testingns
```

# Discovering the bug
In my case it was a simple misconfiguration, can you spot it?  

> Important: be careful with the Calico spec below.  It is likely not what you are expecting.  At the time of writing Calico [vxlan doesn't support IPV6](https://github.com/projectcalico/libcalico-go/issues/996) which is one of the ways to [run Calico on Azure](https://docs.projectcalico.org/networking/vxlan-ipip#encapsulation-types).  Another way is to use [UDR's and host-local ipam](https://docs.projectcalico.org/reference/public-cloud/azure) which requires `--configure-cloud-routes` in the cloud provider which is what is happening above.  I also has the misconfiguration in it.

```yaml
---
# Source: calico/templates/calico-config.yaml
# This ConfigMap is used to configure a self-hosted Calico installation.
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Typha is disabled.
  typha_service_name: "none"
  # Configure the backend to use.
  calico_backend: "none"

  # The CNI network configuration to install on each node.  The special
  # values in this config will be automatically populated.
  # https://docs.projectcalico.org/reference/cni-plugin/configuration#using-host-local-ipam
  cni_network_config: |-
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "datastore_type": "kubernetes",
          "nodename": "__KUBERNETES_NODE_NAME__",
          "mtu": 1500,
          "ipam": {
              "type": "host-local",
              "subnet": "usePodCidr",
          },
          "policy": {
              "type": "k8s"
          },
          "kubernetes": {
              "kubeconfig": "__KUBECONFIG_FILEPATH__"
          }
        },
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        }
      ]
    }
```

Yes, that is comma after `"usePodCidr"` which gave the cryptic error message `network plugin is not ready: cni config uninitialized.`  Fun times with json embedded in yaml :-)  After removing it everything behaved as expected.

## Next steps
I am going to take a look at the K8s/cni code to figure out if it's possible to bubble up better error message as this is obviously a pain point for folks.


