---
layout: post
title: Pre-Warming IIS App in Kubernetes
date: "2018-04-16"
categories:
  - windows containers
  - kubernetes
  - docker
---

Working on a recent project, we were using Kubernetes to host our IIS application in Windows Containers on Windows nodes in the cluster.  We found this worked well but when we went to scale the application we noticed some of our requests would have significant slow down.  We identified that this was the result of the new IIS app entering the Kubernetes service load balancer before the site was warm.  

It is no secret that IIS apps can take a long time to warm up before being able to serve requests.  This behavior can be significant for older applications which load and use lots of libraries.  To ensure that IIS was warmed up before it was added into the service rotation, we configured [Liveliness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) on the `Deployment` manifests.

The [docs on Readiness Probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/#define-readiness-probes) state:

> Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup. In such cases, you don’t want to kill the application, but you don’t want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

This happens to be the exact use case for IIS applications.  It needs to load the entire app pool before it can return a request. Configuring the `Readiness Probe` for a deployment is fairly straight forward:

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: iis-warmup
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: iis-warmup
    spec:
      containers:
      - name: iis-warmup
        image: jsturtevant/whoami-aspnet
        ports:
          - containerPort: 80
        readinessProbe: 
          httpGet:
            path: /
            port: 80
          timeoutSeconds: 3
          periodSeconds: 10
          initialDelaySeconds: 2
        livenessProbe: 
          httpGet:
            path: /
            port: 80
          timeoutSeconds: 3
          periodSeconds: 10
          initialDelaySeconds: 300
      nodeSelector:
        beta.kubernetes.io/os: windows
```

The `ReadinessProbe` above will keep the container out of the service rotation until IIS returns a `200 OK`.  The `LivenessProbe` has been delayed by `300` seconds to ensure that any requests that fail or time out in the initial warm up period don't kill the container prematurely.  Make sure you adjust the `LivenessProbe` for you application start uptime.

# See it in action
I have created a sample Application and Deployment scripts in a sample repository on GitHub.  If you have a mixed workload cluster, great otherwise you can create one with Acs Engine. Once you have a cluster you can use the IIS-prewarm manifest and deploy it to you cluster:

```
curl https://raw.githubusercontent.com/jsturtevant/windows-k8-playground/master/sample-apps/iis-prewarm.yaml
kubectl apply -f iis-prewarm.yaml
```

Next view your deployments and ensure your able to receive requests:

```
kubectl get all

curl http:serviceid
```

Next scale your service and view the events on the new pods:

```
kubectl deployments scale iis-prewarm
kubectl get pods 
```

In the output of `kubectl get pods` you will that the pod is not ready. Viewing the output of the newly added pod you will see that the pod is failing rediness and there for is not in the service rotation:

```
kubectl describe pod <> 
```

After about a minute re-run the command and  you will see the deployment is ready and now you should be able to see both pods in the service rotation:

```
curl http://service
curl http://service
```

