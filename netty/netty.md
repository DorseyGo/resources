# Netty

## 1 Intro

### 1.1 Why Netty?

Netty <font color="#a00"><b>simplifies</b></font> network programming of TCP or UDP servers but you can still access and use low-level APIs because Netty provides high-level abstraction.

- cross platform and compatibility
- Extending
- Scattering and gathering without leak
- squashing the famous epoll

### 1.2 What is Netty?

Netty is a non-blocking, event-driven, networking framework. In reality what this means for you is that Netty uses threads to process IO events.

## 2 Netty at first glance

### 2.1 Writing an Echo Server

Writing a Netty Server consists of two main parts:

- <i>Bootstrapping</i> - Configure server features, such as the threading and port
- <i>Implementing the server handler</i> - Build out the component that contains the business logic, which determines <u>what should happen when a connection is made and data is received</u>.

#### 2.1.1 Bootstrapping the server

You bootstrap a server by creating an instance of the <font color="#aa0">ServerBootstrap</font> class. The instance can then be configured, to set options, such as the port, the threading model/event loop, and the server handler to handle the business logic.

```java
public void start() throws Exception {
    EventLoopGroup group = new NioEventLoopGroup();
    try {
        ServerBootstrap server = new ServerBootstrap();
        server.group(group).channel(NioServerSocketChannel.class).localAddress(new InetSocketAddress(port)).childHandler(new ChannelInitializer<SocketChannel>() {
            public void initChannel(final SocketChannel channel) {
                // add business logic handler
                channel.pipeline().addLast(new EchoServerHandler()); 
            }
        });
        
        ChannelFuture future = server.bind().sync();
        // ....
    }
}
```

Highlight something:

- create a <font color="#aa0">ServerBootstrap</font> instance to bootstrap the server and bind it later
- create and assign the <font color="#aa0">NioEventLoopGroup</font> instances to handle event processing, such as accepting new connections, receiving data, writing data, and so on
- specify the local <font color="#aa0">InetSocketAddress</font> to which the server binds
- setup a <font color="#aa0">ChildHandler</font> that executes the for every accepted connection
- After everything is set up, you call the <font color="#aa0">ServerBootstrap.bind()</font> method to bind the server

#### 2.1.2 Implementing the server/business

To implement the server/business, your <u>channel handler</u> must extend the <font color="#aa0">ChannelInBoundHandlerAdapter</font> class and override the <font color="#aa0">messageReceived</font> method. This method is called every time messages are received, which in this case are bytes.

```java
@Sharable
public class EchoServerHandler extends ChannelInBoundHandlerAdapter {
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        System.out.println("Server received: " + msg);
        ctx.write(msg);
    }
    
    public void channelReadCompleted(ChannelHandlerContext ctx) {
        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER).addListener(ChannelFutureListener.CLOSE);
    }
    
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

Netty uses channel handler to allow a greater separation of concerns, making it easy to add, update, or remove business logic as they evolve. The handler is straightforward and each of its method can be overridden to "hook" into a part of the data life-cycle, but only the <font color="#aa0">channelRead</font> method is required to be overridden.

#### 2.1.3 Intercepting exceptions

Netty's approach to intercepting exceptions makes it easier to handle errors that occur on different threads. Exceptions that were impossible to catch from separate threads are all fed through the same simple, centralized <b>API</b>.

### 2.2 Writing an Echo Client

The client's role includes the following task:

- connect to the server
- writes data
- waits for and receives the exact data back from the server
- closes the connection

#### 2.2.1 Bootstrapping the client

Bootstrapping a client is similar to bootstrapping a server. The client bootstrap, however, accepts both a host and a port to which the client connects, as opposed to only the port.

``` java
public void start() throws Exception {
    EventLoopGroup group = new NioEventLoopGroup();
    try {
        Bootstrap client = new Bootstrap();
        client.group(group).channel(NioSocketChannel.class).remoteAddress(new InetSocketAddress(host, port)).handler(new ChannelInitializer<SocketChannel>() {
            public void initChannel(SocketChannel channel) {
                channel.pipeline().addLast(new EchoClientHandler());
            }
        });
        
        ChannelFuture future = client.connect().sync();
        future.channel().closeFuture().sync();
    } finally {
        client.shutdownGracefully().sync();
    }
}
```

Let's highlight,

- A <font color="#aa0">Bootstrap</font> instance is created to bootstrap the client
- The <font color="#aa0">NioEventLoopGroup</font> instance is created and assigned to handle the event processing, such as creating new connections, receiving data, writing data, etc.
- The remote <font color="#aa0">InetSocketAddress</font> to which the client will connect is specified.
- A handler is set that will be executed once the connection is established
- After everything is setup, the <font color="#aa0">Bootstrap.connect()</font> method is called to connect to the remote peer.

#### 2.2.2 Implementing the client logic

## 3 Netty from the ground up

### 3.1 Channels, Events and IO

> ChannelFuture
>
> Basically ChannelFuture is a placeholder for a result of an operation that is executed in the future. When exactly it is executed depends on many facts and is not easy to "say". 
>
> The only thing can be sure of is that it will be executed and all operations that return a ChannelFuture and belong to the same channel will be executed in the correct order, which is the same order as you executed the methods.

``` mermaid
graph LR
A[incoming request] --> B[Bind EventLoop-n to channel]
C[EventLoopGroup] --Take EventLoop-n --> B
D((EventLoop-n Process IO)) --> B
B --> D
```

when a channel is registered, Netty "binds" that channel to a single <font color="#aa0">EventLoop</font> for the lifetime of that <font color="#aa0">Channel</font>. This is why your application does not need to synchronous on Netty IO operations because all IO for a given <font color="#aa0">Channel</font> will always perform by the same thread. 

> The EventLoop is always bound to a single thread that never changed during its lifetime.

#### 2.2.3 Bootstrapping: What and Why?

Bootstrapping in Netty is the process by which you configure your Netty application. You use a bootstrap when you need to connect a client to some host and port, or bind a server to a given port.

Similarities and differences between two types of Bootstraps (<font color="#aa0">Bootstrap</font>, <font color="#aa0">ServerBootstrap</font>),

|       Similarities        |             Bootstrap              |   ServerBootstrap   |
| :-----------------------: | :--------------------------------: | :-----------------: |
|      Responsible For      | Connects to a remote host and port | Binds to local port |
| Number of EventLoopGroups |                 1                  |          2          |



## 7 Appendix

### 7.1 ByteBuffer

Typical uses of the <font color="#a00">ByteBuffer</font> includes the following,

- writing data to the <font color="#aa0">ByteBuffer</font>
- calling <font color="#aa0">ByteBuffer.flip()</font> to switch from write-mode to reading-mode
- reading data out of the <font color="#aa0">ByteBuffer</font>
- calling either <font color="#aa0">ByteBuffer.clear()</font> or <font color="#aa0">ByteBuffer.compact()</font> 

### 7.2 Working with NIO selectors

A channel represents a connection to an entity capable of performing IO operation such as a file or a socket.

A selector is a NIO component that determines if one or more channels are ready for reading and/or writing, thus a single select selector can be used to handle multiple connections.

To use selectors, you typically complete the following steps,

- create one or more selectors to which opened channels (sockets) can be registered
- when a channel is registered, you specify which events you are interested in listening in. The four available events are,
  - <u>OP_ACCEPT</u> - Operation-set bit for socket-accept operations
  - <u>OP_CONNECT</u> - Operation-set bit for socket-connect operations
  - <u>OP_READ</u> - Operation-set bit for read operations
  - <u>OP_WRITE</u> - Operation-set bit for write operations.
- when the channels are registered, you call the <font color="#aa0">Selector.select()</font> method to block until one of these events occurs
- when the method unblocks, you can obtain all of the <font color="#aa0">SelectionKey</font> instances and do something.

