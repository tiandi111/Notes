Quick note: Connection Multiplexer in etcd
---

- How golang http server works?
The two most important components in http server are Handler(Mux) and Listener.
Let's look at a code segment: <br />
This is a simplified version of server.Serve. It is basically a for loop that
continuously accept incoming connections and start a new goroutine for each one
to serve.
```go
func (srv *Server) Serve(l net.Listener) error {
	for {
		rw, e := l.Accept()
		if e != nil {
			handleError(e)
		}
		c := srv.newConn(rw)
		go c.serve(ctx)
	}
}
```
Within c.serve, there's another for loop that continuously read request and
```go
func (c *conn) serve(ctx context.Context) {
	for {
		w, err := c.readRequest(ctx)
		if err != nil {
			handleError(err)
		}
		req := w.req
		serverHandler{c.server}.ServeHTTP(w, w.req)
	}
}
```
handle it with the handler attached to the server just like below:
```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	handler.ServeHTTP(rw, req)
}
```
In goland http package, a struct called mux implements Handler, it simply routes
the incoming request to a corresponding handler and serve it.
```go
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	if r.RequestURI == "*" {
		if r.ProtoAtLeast(1, 1) {
			w.Header().Set("Connection", "close")
		}
		w.WriteHeader(StatusBadRequest)
		return
	}
	h, _ := mux.Handler(r)
	h.ServeHTTP(w, r)
}
```

- What is a connection-level multiplexer?
A connection-level multiplexer is a one that multiplexes connections with different
communication protocol, such as grpc, http, trpc, to different servers on a single
port.

- How etcd do connection-level multiplexer?
etcd uses the soheilhy/cmux package to do connection multiplexer. The basic idea
behind this multiplexer is to build a gateway listener in front of other listeners.
When a request comes, it "sniffs" the connection by look at the first several bytes
to decide which protocol it is using and redirect it to the right server. The github
address is here: https://github.com/soheilhy/cmux.

This is the simplified Serve() method of connection-level multiplexer. It simply receives
incoming connection from a root listener and dispatch it in serve().
```go
func (m *cMux) Serve() error {
	for {
		c, err := m.root.Accept()
		if err != nil {
			if !m.handleErr(err) {
				return err
			}
			continue
		}

		wg.Add(1)
		go m.serve(c, m.donec, &wg)
	}
}
```
Again, simplified serve() method on cMux, it chooses a matched listener and dispatches the
connection by a channel.
```go
func (m *cMux) serve(c net.Conn, donec <-chan struct{}, wg *sync.WaitGroup) {
	muc := newMuxConn(c)
	for _, sl := range m.sls {
		for _, s := range sl.ss {
			matched := s(muc.Conn, muc.startSniffing())
			if matched {
				muc.doneSniffing()
				if m.readTimeout > noTimeout {
					_ = c.SetReadDeadline(time.Time{})
				}
				select {
				case sl.l.connc <- muc:
				case <-donec:
					_ = c.Close()
				}
				return
			}
		}
	}
}
```
This is the actual listener for each server, it receives connections from the channel.
```go
func (l muxListener) Accept() (net.Conn, error) {
	c, ok := <-l.connc
	if !ok {
		return nil, ErrListenerClosed
	}
	return c, nil
}
```