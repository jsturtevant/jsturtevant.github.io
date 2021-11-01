---
layout: post
title: Running Windows Unit tests for Kubenretes on Windows
date: "2021-11-01"
categories:
  - kubernetes
  - windows
  - golang
  - goland
---


Most of the [kubernetes build tooling](https://github.com/kubernetes/kubernetes/tree/master/hack) doesn't work on Windows but we do have unit tests written for Windows components.  These unit tests must run on  a Windows machine. So how do you run them if the build tooling doesn't work?  

## Manually
From a Windows machine you can manually run them:

```
go test . -mod=mod -run listContainerNetworkStats
```

If you want to build build them on Linux then run them on Windows. 

On Linux, `cd` to the folder where your tests are and build a test executable:

```
cd pkg\kubelet\stats\
GOOS=windows go test -c .
```

Then on Windows, the test executable can then be run (where `listContainerNetworkStats` is the name of the test to focus on):

```
#copy test executable to windows and run:
stats.test.exe -test.run listContainerNetworkStats
```

## Goland
To enable running them in Goland on Windows:

![](https://i.imgur.com/qojCatL.png)

Then you can run the test (and debug!) via the UI.

![](https://i.imgur.com/7kHtz01.png)
## VSCode
To enable running them in VS Code on Windows add the following to your settings file.  It has to be done in the file and as of the writing can't be configured in the UI.

```
{
    "go.testFlags": [
        "-mod=mod"
    ]
}
```

Then you can run the test (and debug!) via the UI.

![](https://i.imgur.com/6h1e6EH.png)
