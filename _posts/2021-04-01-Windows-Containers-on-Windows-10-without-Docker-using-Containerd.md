---
layout: post
title: Windows Containers on Windows 10 without Docker (using Containerd)
date: "2021-04-01"
categories:
  - windows containers
  - docker
  - containerd
  - kubernetes
---

I have been working on making [containerd](https://containerd.io/docs/) work well on Windows with Kubernetes.  I needed to do some local dev on containerd so I started to configure my local machine.  I found lots of information here and there but nothing comprehensive so I wrote down my steps. 

Note there are a few limitations with containerd (so you might not want to fully uninstall Docker). For instance, containerd doesn't support building containers and you won't be able to use the native Linux containers functionality (though there is some early support for LCOW).

Let's get started!

## Get Containerd

Get and install containerd:

```powershell
curl.exe -LO https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-windows-amd64.tar.gz

#install
tar xvf containerd-1.4.4-windows-amd64.tar.gz
mkdir -force "C:\Program Files\containerd"
mv ./bin/* "C:\Program Files\containerd"

"C:\Program Files\containerd\containerd.exe" config default | Out-File "C:\Program Files\containerd\config.toml" -Encoding ascii
```

Next for performance tell Windows defender to ignore it and start the service:

```powershell
Add-MpPreference -ExclusionProcess "$Env:ProgramFiles\containerd\containerd.exe"

.\containerd.exe --register-service
Start-Service containerd
```

## Setting up network

Unlike Docker, Containerd doesn't attach the pods to a network directly.  It uses a CNI (container networking interface) plugin to set up the networking.  We will use the windows nat plugin for our local dev env.  Not this is likely the plugin you would want to use in a kubernetes cluster, for that you should look at Calico, Antrea, or many of the cloud specific ones that are avalaible.  For our dev environment NAT will work just fine.

Create the folder for cni binaries and configuration.  The path `C:\Program Files\containerd\cni\bin` is the default location for containerd.

```powershell
mkdir -force "C:\Program Files\containerd\cni\bin"
mkdir -force "C:\Program Files\containerd\cni\conf"
```

Get the nat binaries:

```powershell
curl.exe -LO https://github.com/microsoft/windows-container-networking/releases/download/v0.2.0/windows-container-networking-cni-amd64-v0.2.0.zip
Expand-Archive windows-container-networking-cni-amd64-v0.2.0.zip -DestinationPath "C:\Program Files\containerd\cni\bin" -Force
```


Create a nat network.  For nat network it must have the name `nat`

```powershell
 curl.exe -LO https://raw.githubusercontent.com/microsoft/SDN/master/Kubernetes/windows/hns.psm1
ipmo ./hns.psm1

$subnet="10.0.0.0/16" 
$gateway="10.0.0.1"
New-HNSNetwork -Type Nat -AddressPrefix $subnet -Gateway $gateway -Name "nat"
```

Set up the containerd network config using the same gateway and subnet.

```powershell
@"
{
    "cniVersion": "0.2.0",
    "name": "nat",
    "type": "nat",
    "master": "Ethernet",
    "ipam": {
        "subnet": "$subnet",
        "routes": [
            {
                "gateway": "$gateway"
            }
        ]
    },
    "capabilities": {
        "portMappings": true,
        "dns": true
    }
}
"@ | Set-Content "C:\Program Files\containerd\cni\conf\0-containerd-nat.conf" -Force
```

## Creating containers

There are two main ways to interact with containerd: ctr and crictl.  ctr is great for simple testing and crictl is used to interact with containerd in the same way that kubernetes works.

### CTR

Ctr was already installed when you pulled the binaries and installed containerd.  It has a similiar interface to docker but it does vary some since you are working at a different level.

There is a bug with ctr that doesn't support multi-arch images on Windows ([PR](https://github.com/containerd/containerd/pull/5298) to fix it) so we will use the specific image that matches our operating system.  I am running Windows 10 20H2 (build number 19042) so will use that tag for the image.

```
# find your version
cmd /c ver
Microsoft Windows [Version 10.0.19042.867]

cd "C:\Program Files\containerd\"
./ctr.exe pull mcr.microsoft.com/windows/nanoserver:20H2 
./ctr.exe run -rm  mcr.microsoft.com/windows/nanoserver:20H2 test cmd /c echo hello
hello
```

### Create a pod via crictl

Using CRI api can be alittle cumbersome.  It requires doing each step independently.  

Get the crictl and install it to a directly:

```powershell
curl.exe -LO https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.20.0/crictl-v1.20.0-windows-amd64.tar.gz                          
tar xvf crictl-v1.20.0-windows-amd64.tar.gz
```

Set up defaults for `crictl` that will connect to containerd's default named pipe so it doesn't need to be set for each call with `crictl`.

```powershell
@"
runtime-endpoint: npipe://./pipe/containerd-containerd
image-endpoint: npipe://./pipe/containerd-containerd
timeout: 10
#debug: true
"@ | Set-Content "crictl.yaml" -Force
```

Pull an image

```powershell
./crictl pull k8s.gcr.io/pause:3.4.1
```

Create the sandbox, container and run it (told you its a bit cumbersome).

```powershell
$POD_ID=(./crictl runp .\pod.json)
$CONTAINER_ID=(./crictl create $POD_ID .\container.json .\pod.json)
./crictl start $CONTAINER_ID
```

Finally exec in and check the pod has an IP address from the network we crated earlier.

First lets take a look at the network and endpoint that was created for the pod:

```powershell
get-hnsnetwork | ? Name -Like "nat" | select name, id, subnets

Name ID                                   Subnets
---- --                                   -------
nat  D95007EB-27C6-4EED-87FC-402187447637 {@{AdditionalParams=; AddressPrefix=10.0.0.0/16; Flags=0; GatewayAddress=1...

Get-HnsEndpoint | ? virtualnetworkname -like nat | select name, id, ipaddress, gatewayaddress

Name                                                                 ID                                   IPAddress    GatewayAddress
----                                                                 --                                   ---------    --------------
92c0776a2fa06e2ce0a074b350146ed9e97bb2dafdb5bf0b8f8c55c8d4020f00_nat 50f2bd95-90c2-4234-8d82-ef45c841dfb4 10.0.113.113 10.0.0.1
```

Now exec into the container and look at the network config

```powershell
./crictl exec -i -t $CONTAINER_ID cmd

C:\>ipconfig
Windows IP Configuration
Ethernet adapter vEthernet (92c0776a2fa06e2ce0a074b350146ed9e97bb2dafdb5bf0b8f8c55c8d4020f00_nat):

Connection-specific DNS Suffix  . : home
Link-local IPv6 Address . . . . . : fe80::3034:e49c:68b4:99ac%88
IPv4 Address. . . . . . . . . . . : 10.0.113.113 
Subnet Mask . . . . . . . . . . . : 255.255.0.0  
Default Gateway . . . . . . . . . : 10.0.0.1 
```

You will note the endpoint is the name as the Ethernet Adapter and the ipddresss and default gateway match.

## Helpful links that I used to compile this walk through

- https://github.com/kubernetes-sigs/cri-tools/blob/master/docs/crictl.md
- https://kubernetes.io/docs/tasks/debug-application-cluster/crictl/
- https://github.com/kubernetes-sigs/sig-windows-tools/blob/master/kubeadm/scripts/Install-Containerd.ps1
- https://github.com/containerd/containerd/blob/master/script/setup/install-cni-windows

