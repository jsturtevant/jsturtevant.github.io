---
layout: post
title: Attaching a debugger to a Windows Containerd shim
date: "2023-03-25"
categories:
  - containerd
  - golang
  - debuggers
---

Containerd provides a mechanism that allows runtime authors to integrate with containerd via the [shim api](https://github.com/containerd/containerd/blob/main/runtime/v2/README.md).  The general way this works is that containerd invokes a binary on the host then communicates via an IPC channel that works for a the operating system (namedpipes on windows).  There is a lot of details in the readme but that is the gist.  

It can be tricky to debugging the runtime is that since it is a new process that splits off from the main containerd process.  There is a lot of setup that happens when the shim initially launches like creating the rootfs or configuring and launching the container process. It is hard to attach a debugger at that critical time.  Once the process is running especially, if it long running, you can attach a debugger just fine but there is a lot of details that happen in the first few moments of the shim starting up.

I came across a great way of handling this and I just have to share it.  This is in the Windows Containerd Shim but could be used for an Windows process that is similar. 

> Important note: This is for developer debugging do not do this in production. It will stop all shims in there tracks.

For the Windows shim you can set an env variable, then start containerd:

```powershell
$env:CONTAINERD_SHIM_RUNHCS_V1_WAIT_DEBUGGER = "true"
.\containerd.exe -c .\config.toml -l trace
time="2023-03-25T14:32:41.237855400-07:00" level=info msg="starting containerd" revision=63e45eb5d8b3949195b5332876b591d88977f3b9.m version=v1.7.0-34-g63e45eb5d.m
...more startup output...
time="2023-03-25T14:35:25.845627800-07:00" level=info msg="Start event monitor"
time="2023-03-25T14:35:25.845740900-07:00" level=info msg="Start snapshots syncer"
time="2023-03-25T14:35:25.845740900-07:00" level=info msg="Start cni network conf syncer for default"
time="2023-03-25T14:35:25.845740900-07:00" level=info msg="Start streaming server
```

Once containerd is running you can start any container. I will show it with ctr.exe but it can be started from any interface like nerdctl or crictl.

```powershell
ctr run --rm gcr.io/k8s-staging-e2e-test-images/busybox:1.29-2-windows-amd64-ltsc2022 win sleep 100
```

You will immediately see the process hang, and it is not because of the `sleep`.  If you go and look at the containerd logs you should see:

```powershell
time="2023-03-25T14:38:43.235031500-07:00" level=info msg="Halting until signalled" event="Global\\debugger-17216"
```

The shim was launched and it is waiting until you tell it to continue! The last few numbers are the process id:

```powershell
ps *shim*
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    107       9    18956      13108       0.02  17216   1 containerd-shim-runhcs-v1
```

So now, before all the important things happen, you can attached a debugger.  You can [attach it via console](https://www.jamessturtevant.com/posts/Using-the-Go-Delve-Debugger-from-the-command-line/) or do it remotely and then use something like [goland](https://www.jetbrains.com/help/go/go-remote.html) or [vscode](https://github.com/golang/vscode-go/wiki/debugging#remote-debugging) to attached to the remote debugger.

```powershell
dlv --listen=:2345 --headless=true --api-version=2 attach 17216 
```

Now you will notice that even though the debugger might be attached, the process hasn't continued.  This is because it is waiting for an explicit event to continue.  To do this you can use a tool [docker-signal](https://github.com/moby/docker-signal) which is a small program that sends the expected event. Clone the repo and run the tool.  

> Don't forget to set a breakpoint in the debugger first!

```powershell
go run .\docker-signal.go --debugger -pid 34344
```

And there you have it!  Attaching the debugger before all the critical code runs can save tons of time figuring out that critical bug.

The containerd shim does need to be built with debugging info. To do that clone the code, build it and copy it to the containerd path:

```powershell
cd .\projects\hcsshim\
go build -gcflags "all=-N -l" .\cmd\containerd-shim-runhcs-v1\
cp containerd-shim-runhcs-v1.exe ../path/to/containerd/folder
```

## Looking under the covers

This is great, but how does it work? The important part is during the `CreateTask`: 

```golang
func (s *service) createInternal(ctx context.Context, req *task.CreateTaskRequest) (*task.CreateTaskResponse, error) {
	setupDebuggerEvent()
    ...
}
```

The `Create` task is one of the first calls that containerd makes and is before all the critical code that creates the layers and containers in Windows.  Let's take a look at [setup DebuggerEvent](https://github.com/microsoft/hcsshim/blob/dd669924dbbfda544ebafe3f94a3b5d2a0e4412f/cmd/containerd-shim-runhcs-v1/serve.go#L315):

```golang
func setupDebuggerEvent() {
	if os.Getenv("CONTAINERD_SHIM_RUNHCS_V1_WAIT_DEBUGGER") == "" {
		return
	}
	event := "Global\\debugger-" + fmt.Sprint(os.Getpid())
	handle, err := createEvent(event)
	if err != nil {
		return
	}
	logrus.WithField("event", event).Info("Halting until signalled")
	_, _ = windows.WaitForSingleObject(handle, windows.INFINITE)
}
```

If that env variable isn't set then skip this part, otherwise create a [Windows Event](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-createeventw) and wait for  the event.  `WaitForSingleObject` in Windows will put the current threat to sleep until the event is signaled. 

And you guessed it, signaling the event is what [docker-signal](https://github.com/moby/docker-signal/blob/master/docker-signal.go#L65-L71) does!

```
	ev := fmt.Sprintf("Global\\%s-%s", key, fmt.Sprint(pid))
	h2, _ := OpenEvent(EVENT_MODIFY_STATUS, false, ev)
	if h2 == 0 {
		fmt.Printf("Could not open event. Check PID %d is correct and is running.\n", pid)
		return
	}
	PulseEvent(h2)
```

## Conclusion

This is a nifty little trick to use when you have complex processes that interact across boundaries. Even though this example is for Windows Container shim, it could be done in an process where it is difficult to start up the debugger and it could be in any language!

## Bonus content

This type of trick can be used for all sorts of stuff. Another example is containerd where we use the signal to dump the stacks at any time the event is received.  Get creative and have fun!

https://github.com/containerd/containerd/blob/f7f2be732159a411eae46b78bfdb479b133a823b/cmd/containerd/command/main_windows.go#LL70