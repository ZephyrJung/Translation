### Fundamental

- [Echo](http://netty.io/4.1/xref/io/netty/example/echo/package-summary.html)‐the very basic client and server
- [Discard](http://netty.io/4.1/xref/io/netty/example/discard/package-summary.html)‐see how to send an infinite data stream asynchronously without flooding the write buffer
- [Uptime](http://netty.io/4.1/xref/io/netty/example/uptime/package-summary.html)‐implement automatic reconnection mechanism

### Text protocols

- [Telnet](http://netty.io/4.1/xref/io/netty/example/telnet/package-summary.html)‐a classic line-based network application
- [Quote of the Moment](http://netty.io/4.1/xref/io/netty/example/qotm/package-summary.html)‐broadcast a UDP/IP packet
- [SecureChat](http://netty.io/4.1/xref/io/netty/example/securechat/package-summary.html)‐an TLS-based chat server, derived from the Telnet example

### Binary protocols

- [ObjectEcho](http://netty.io/4.1/xref/io/netty/example/objectecho/package-summary.html)‐exchange serializable Java objects
- [Factorial](http://netty.io/4.1/xref/io/netty/example/factorial/package-summary.html)‐write a stateful client and server with a custom binary protocol
- [WorldClock](http://netty.io/4.1/xref/io/netty/example/worldclock/package-summary.html)‐rapid protocol protyping with Google Protocol Buffers integration

### HTTP

- [Snoop](http://netty.io/4.1/xref/io/netty/example/http/snoop/package-summary.html)‐build your own extremely light-weight HTTP client and server
- [File server](http://netty.io/4.1/xref/io/netty/example/http/file/package-summary.html)‐asynchronous large file streaming in HTTP
- Web Sockets([Client](http://netty.io/4.1/xref/io/netty/example/http/websocketx/client/package-summary.html) &[Server](http://netty.io/4.1/xref/io/netty/example/http/websocketx/server/package-summary.html))‐add a two-way full-duplex communication channel to HTTP using Web Sockets
- SPDY([Client](http://netty.io/4.1/xref/io/netty/example/spdy/client/package-summary.html) &[Server](http://netty.io/4.1/xref/io/netty/example/spdy/server/package-summary.html))‐implement [SPDY](http://en.wikipedia.org/wiki/SPDY) protocol
- [CORS demo](http://netty.io/4.1/xref/io/netty/example/http/cors/package-summary.html)‐implement [cross-origin resource sharing](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)

### Advanced

- [Proxy server](http://netty.io/4.1/xref/io/netty/example/proxy/package-summary.html)‐write a highly efficient tunneling proxy server
- [Port unification](http://netty.io/4.1/xref/io/netty/example/portunification/package-summary.html)‐run services with different protocols on a single TCP/IP port

### UDT

- [Byte streams](http://netty.io/4.1/xref/io/netty/example/udt/echo/bytes/package-summary.html)‐use [UDT](http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol) in TCP-like byte streaming mode
- [Message flow](http://netty.io/4.1/xref/io/netty/example/udt/echo/message/package-summary.html)‐use [UDT](http://en.wikipedia.org/wiki/UDP-based_Data_Transfer_Protocol) in UDP-like message delivery mode
- [Byte streams in symmetric peer-to-peer rendezvous connect mode](http://netty.io/4.1/xref/io/netty/example/udt/echo/rendezvousBytes/package-summary.html)
- [Message flow in symmetric peer-to-peer rendezvous connect mode](http://netty.io/4.1/xref/io/netty/example/udt/echo/rendezvous/package-summary.html)