---
layout: post
title: Running Kubernetes Minikube on Windows 10 with WSL
date: "2017-12-15"
categories:
  - kubernetes
  - wsl
---

Sometimes you want to be able to deploy and develop applications locally with out having to spin up an entire cluster.  Setting up Minikube on Windows 10 hasn't been the easiest thing to do but with the help of a colleague, [Noel Bundick](https://www.noelbundick.com/) and [GitHub issues](https://github.com/kubernetes/minikube/issues/2131), I got it working this week so this post is for me in the future when I can't remember how i did it.:-).

## Install Minikube
This part is pretty easy if you use [Chocolately](https://chocolatey.org/) (not using Chocolatly?  [Check out why you should]({{ site.url }}/posts/Chocolatey-And-Boxstarter/)).  Alternatively you can [download it and add it to your path](https://github.com/kubernetes/minikube/releases).

```bash
choco install minikube
```

## Create a Virtual Switch in Hyper-V
This is the extra step you need to do to get Hyper-V to work with minikube.  Open a Powershell prompt and type:

```powershell
# get list of network adapter to attach to
Get-NetAdapter  

#output
Name                      InterfaceDescription                    ifIndex Status          LinkSpeed
----                      --------------------                    ------- ------          ---------
vEthernet (minikube)      Hyper-V Virtual Ethernet Adapter #3          62 Up              400 Mbps
Network Bridge            Microsoft Network Adapter Multiplexo...      46 Up              400 Mbps
vEthernet (nat)           Hyper-V Virtual Ethernet Adapter #2          12 Up              10 Gbps
vEthernet (Default Swi... Hyper-V Virtual Ethernet Adapter             13 Up              10 Gbps
Bluetooth Network Conn... Bluetooth Device (Personal Area Netw...      23 Disconnected    3 Mbps
Ethernet                  Intel(R) Ethernet Connection (4) I21...       9 Disconnected    0 bps
Wi-Fi                     Intel(R) Dual Band Wireless-AC 8265          14 Up              400 Mbps

#Create the switch
New-VMSwitch -name minikube  -NetAdapterName <your-network-adapter-name> -AllowManagementOS $true  
```

## Create minikube
From the Powershell prompt in Windows, create minikube using the switch you just created:

```powershell
minikube start --vm-driver hyperv --hyperv-virtual-switch minikube
```

Minikube adds the configuration to your `.kube/config` file upon successful creation so you should be able to connect to the minikube from the *powershell* prompt using `kubectl` if you have it [installed on windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-chocolatey-on-windows):

```powershell
kubectrl get nodes

#output
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    18h       v1.8.0
```

## Using WSL to talk to minikube
I mostly use WSL as my command prompt in Windows these days which means I have `kubectl`, `helm` and my other tools all installed there.  Since we just installed minikube on windows, the `.kube/config` file was created on the windows side at `C:\Users\<username>\.kube\config`.  To get `kubectl` to work we will need to add the configuration to our `.kube/config` on WSL at `/home/<bash-user-name>/.kube`.  

> Note: the following might vary depending on your existing `.kube/config` file and set up.  Check out [sharing cluster access on kubernetes docs for more info](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) and alternative ways to configure.  

To see the values created on for you *windows environment* you can run `kubectl context view` from your *powershell* prompt.  Use those values for the minikube entries below.

From your WSL terminal add the minikube context info:

```bash
kubectl config set-cluster minikube --server=https://<minikubeip>:port --certificate-authority=/mnt/c/Users/<windows-user-name>/.minikube/ca.crt
kubectl config set-credentials minikube --client-certificate=/mnt/c/Users/<windows-user-name>/.minikube/cert.crt --client-key=/mnt/c/Users/<windows-user-name>/.minikube/client.key
kubectl config set-context minikube --cluster=minikube --user=minikube
```

This points the context at the cert files minikube created on Windows.  To verify you have set the values correctly view the context in WSL (if you have other contexts if might look slightly different):

```bash
kubectl config view

#output
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /mnt/c/Users/<windows-user-name>/.minikube/ca.crt
    server: https://<minikubeip>port
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /mnt/c/Users/<windows-user-name>/.minikube/client.crt
    client-key: /mnt/c/Users/<windows-user-name>/.minikube/client.key
```

Now set your current context to minikube and try connecting to your minikube instance:

```bash
kubectl config use-context minikube

kubectl get nodes

#output
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    20h       v1.8.0
```

## Limitations
I can use kubectl as I would with any other cluster but I have found that the can't run the minikube commands from WSL.  I have to go back to my Windows prompt to run commands like `minikube service <servicename> --url`