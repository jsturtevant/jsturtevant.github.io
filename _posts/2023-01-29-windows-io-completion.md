---
layout: post
title: Windows I/O completion - One little trick
date: "2023-01-29"
categories:
  - windows
  - rust
  - golang 
---

I've been learning how to deal with [I/O Completion ports](https://learn.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports) for my latest project and found a few libraries that manage it all for me but I was getting strange behavior so I ended up having to dig deep enough to understand what was happening. I didn't find a really clear post so here is my attempt.

When I was reading some of the code I found they were all has slightly different ways of accomplishing the way to detect a completed I/O call. The two libraries I was referencing were [Rust's Mio](https://github.com/tokio-rs/mio) crate and [go's winio](https://github.com/microsoft/go-winio).

Understanding that they were accomplishing the same task in different ways was key:

- Winio library is treating the read as a synchronous call.  So when you call `Read` or `Write` on then it will issue the read call, then wait till the async operation completes.  These means if you wish to use this as an async call you should do it on a Go Routine.
- Mio library creates an [event loop](https://en.wikipedia.org/wiki/Event_loop) and the processing should be handled out side of the call.  It also converts [Windows IO *completion* into a *readiness* signal using an internal buffer](https://github.com/tokio-rs/mio/blob/fa4e4b3c58af76909d047abd81d3eaca1d8c5736/src/sys/windows/named_pipe.rs#L37-L41).

## I/O completion's one little trick

Those are two key differences in the way each library approaches doing I/O but I was still confused as to how the program "wakes" back up after the I/O completes.  Let's take a look at the [winio code](https://github.com/microsoft/go-winio/blob/650a2e464c7458a1d355d7bb5affad1eec1bfbb6/file.go#L167-L178) that returns after the system finished the async call to `GetQueuedCompletionStatus`. Note that the system call to `getQueuedCompletionStatus` will suspend the thread that calls it. 

```
// ioOperation represents an outstanding asynchronous Win32 IO.
type ioOperation struct {
	o  syscall.Overlapped
	ch chan ioResult
}

func ioCompletionProcessor(h syscall.Handle) {
	for {
		var bytes uint32
		var key uintptr
		var op *ioOperation
		err := getQueuedCompletionStatus(h, &bytes, &key, &op, syscall.INFINITE)
		if op == nil {
			panic(err)
		}
		op.ch <- ioResult{bytes, err}
	}
}
```

What is going on here?  How does the Operating System call know how to fill in an ` op *ioOperation` and how can we then pass data into the channel?  

To figure this out we need to see how I/O is "prepared" and then invoked. To prepare the I/O we create an I/O operation:

```
func (f *win32File) prepareIO() (*ioOperation, error) {
	f.wgLock.RLock()
	if f.closing.isSet() {
		f.wgLock.RUnlock()
		return nil, ErrFileClosed
	}
	f.wg.Add(1)
	f.wgLock.RUnlock()
	c := &ioOperation{}
	c.ch = make(chan ioResult)
	return c, nil
}
```

Then we issue the `Read` and wait for it to complete async:

```
	var bytes uint32
	err = syscall.ReadFile(f.handle, b, &bytes, &c.o)
	n, err := f.asyncIO(c, &f.readDeadline, bytes, err)
	runtime.KeepAlive(b)
```

Inside the `asyncIO we find we are waiting for the channel to be filled:

```
	var r ioResult
	select {
	case r = <-c.ch:
		err = r.err
		if err == syscall.ERROR_OPERATION_ABORTED { //nolint:errorlint // err is Errno
			if f.closing.isSet() {
				err = ErrFileClosed
			}
		} else if err != nil && f.socket {
			// err is from Win32. Query the overlapped structure to get the winsock error.
			var bytes, flags uint32
			err = wsaGetOverlappedResult(f.handle, &c.o, &bytes, false, &flags)
		}
	case <-timeout:
	    ...snip...
		}
	}
```

If you read the rest of the code you will not find that channel used anywhere!  But as you might have guessed by now that channel that we saw in the `ioCompletionProcessor` is the same!  How do the two channels get linked together?  

The key is a little trick that is used extensively when working with I/O completion ports.  When calling `getQueuedCompletionStatus` we are passing a pointer to the structure [Overlapped](https://learn.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-overlapped).  The struct we passed look is actually a wrapper:

```
type ioOperation struct {
	o  syscall.Overlapped
	ch chan ioResult
}
```

Since we prepared set up the channel during `prepareio` then passed the pointer to the `Read` sys call **and** the OS only fills in the bits for the `Overlapped` struct when we get the notification that the thread is unsuspended we can have a pointer the the struct that we prepared.  Then we can pass the value through the channel (which is waiting in the `asyncIO`) and the read completes!

This little trick is also used the in the Mio project but slightly differently.  Since the Mio project as created an event loop it doesn't actually wait for the read it just needs to know the event it is associated too, The read will happen at another time.  So instead the structure looks a little different but the same trick is used:

```
#[repr(C)]
pub(crate) struct Overlapped {
    inner: UnsafeCell<OVERLAPPED>,
    pub(crate) callback: fn(&OVERLAPPED_ENTRY, Option<&mut Vec<Event>>),
}
```

In this case they've make it a generic callback function that can be filled with anything.  In other cases you might just have some basic information in and not a call back or channel.  It really is up to your use case.

## Conclusion
It took me awhile to figure what how this call came together and it was hard to find it explicitly called out anywhere. Hopefully this helps someone who is struggling to figure out the "one small trick" being used here. 

I did find eventually find this in a few resources on the topic. I highly recommend reading the following which go over the details in much more detail:

- https://cfsamsonbooks.gitbook.io/epoll-kqueue-iocp-explained/
- https://dschenkelman.github.io/2013/10/29/asynchronous-io-in-c-io-completion-ports/
- https://leanpub.com/windows10systemprogramming (Chapter 11 File and Device I/O)
