Go Listener and Linux Socket 
---

### net.Listen()

Entry Point:
```go
func Listen(network, address string) (Listener, error) {
	var lc ListenConfig
	return lc.Listen(context.Background(), network, address)
}
```
We can listen on tcp or unix:
```go
func (lc *ListenConfig) Listen(ctx context.Context, network, address string) (Listener, error) {
	addrs, err := DefaultResolver.resolveAddrList(ctx, "listen", network, address, nil)
	...
	la := addrs.first(isIPv4)
	switch la := la.(type) {
	case *TCPAddr:
		l, err = sl.listenTCP(ctx, la)
	case *UnixAddr:
		l, err = sl.listenUnix(ctx, la)
	default:
	    ...
    }
	...
}
```
Let's do tcp first:
- Listen on TCP
```go
func (sl *sysListener) listenTCP(ctx context.Context, laddr *TCPAddr) (*TCPListener, error) {
	fd, err := internetSocket(ctx, sl.network, laddr, nil, syscall.SOCK_STREAM, 0, "listen", sl.ListenConfig.Control)
	if err != nil {
		return nil, err
	}
	return &TCPListener{fd: fd, lc: sl.ListenConfig}, nil
}
```
```go
func internetSocket(ctx context.Context, net string, laddr, raddr sockaddr, sotype, proto int, mode string, ctrlFn func(string, string, syscall.RawConn) error) (fd *netFD, err error) {
	...
	return socket(ctx, net, family, sotype, proto, ipv6only, laddr, raddr, ctrlFn)
}
```
The core of TCPListener is a socket file descriptor fd, let's look at how it is created,
```
func socket(ctx context.Context, net string, family, sotype, proto int, ipv6only bool, laddr, raddr sockaddr, ctrlFn func(string, string, syscall.RawConn) error) (fd *netFD, err error) {
	s, err := sysSocket(family, sotype, proto)
	    ...
	fd, err = newFD(s, family, sotype, net)
        ...
    
    switch (some conditions) {
        case syscall.SOCK_STREAM, syscall.SOCK_SEQPACKET:

            fd.listenStream(laddr, listenerBacklog(), ctrlFn)

        case syscall.SOCK_DGRAM

            fd.listenDatagram(laddr, ctrlFn)

        default:
            ...
    }

    fd.dial(ctx, laddr, raddr, ctrlFn)

	return fd, nil
}
```
we see that we first do a system call socket(), then we listen on that socket by different ways.
1. Stream<br/>
There are four main steps when listen on stream mode, they are system bind, system
listen, fd init and setAddr. In short, system bind binds the socket with the given
address, system listen enable the socket to passively accept incoming connections,
fd.init() init the given fd, we will go into some details soon on this, and setAddr
assigns the real address to fd and set the finalizer which is used to finalize the 
given interface when garbage collector decide to clear it.  
```
func (fd *netFD) listenStream(laddr sockaddr, backlog int, ctrlFn func(string, string, syscall.RawConn) error) error {
	...
	var lsa syscall.Sockaddr
	lsa, err = laddr.sockaddr(fd.family)
	...
	syscall.Bind(fd.pfd.Sysfd, lsa)
        ...
	listenFunc(fd.pfd.Sysfd, backlog)
        ...
	fd.init()  
        ...
	lsa, _ = syscall.Getsockname(fd.pfd.Sysfd)
	fd.setAddr(fd.addrFunc()(lsa), nil)
	return nil
}
```
Let's look at fd.init() here:
```go
func (fd *netFD) init() error {
	return fd.pfd.Init(fd.net, true)
}
```
pfd is of type FD in poll package, according to the comment, it is a file descriptor
that net and os package use it to represent a network connection or OS file. Its 
init function is shown below.
```go
func (fd *FD) Init(net string, pollable bool) error {
	// We don't actually care about the various network types.
	if net == "file" {
		fd.isFile = true
	}
	if !pollable {
		fd.isBlocking = 1
		return nil
	}
	err := fd.pd.init(fd)
	if err != nil {
		// If we could not initialize the runtime poller,
		// assume we are using blocking mode.
		fd.isBlocking = 1
	}
	return err
}
```
The main step is to init pollDesc. 
```go
func (pd *pollDesc) init(fd *FD) error {
	serverInit.Do(runtime_pollServerInit)
	ctx, errno := runtime_pollOpen(uintptr(fd.Sysfd))
	if errno != 0 {
		if ctx != 0 {
			runtime_pollUnblock(ctx)
			runtime_pollClose(ctx)
		}
		return errnoErr(syscall.Errno(errno))
	}
	pd.runtimeCtx = ctx
	return nil
}
```