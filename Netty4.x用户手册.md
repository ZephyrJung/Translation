# 4.x用户手册

### 前言

#### 问题

如今，我们使用目标通用的应用或库来进行相互间的通讯。例如，我们常使用HTTP客户端库来检索来自服务端的信息，并通过网络服务（web service）来进行一个远程过程调用。然而，一个通用目标的协议或者它的实现有时候并不能尽善尽美。就好像我们不会用一个通用目标的HTTP服务器来交换大型文件，电子邮件信息，以及近乎实时的信息如金融信息和多人游戏数据。这些都要求针对特定的目标来实现高度优化的协议。例如，你可能想要实现一个HTTP服务器能够针对基于Ajax聊天应用，流媒体或大型文件传输来优化。你甚至会想设计并实现一个全新的协议来为自己的需求精确的量身定做。另一个不可避免的场景是处理历史遗留的专用协议来保证和老系统的互通性。这个场景的重点在于如何能为目标应用在不牺牲稳定和性能的前提下快速的实现一个协议。

### 解决方案

Netty项目是一个提供异步时间驱动网络应用框架和快速开发可维护的高性能高扩展性服务端和客户端协议工具集的成果。

换句话说，Netty是一个NIO客户端服务端框架，它使得快速而简单的开发像服务端客户端协议的网络应用成为了可能。它它极大的简化并流线化了如TCP和UDP套接字服务器开发的网络编程。

“快速且简便”不意味着目标应用将容忍维护性和性能上的问题。Netty在吸取了大量协议实现（如FTP，SMTP，HTTP以及各种二进制，基于文本的传统协议）的经验上进行了精心的设计。由此，Netty成功找到了一个无需折衷妥协而让开发、性能、稳定性和灵活性相互协调的方法。

一些用户可能已经找到了其他的生成有同样有点的网络应用框架，并想知道Netty与这些有什么不同。答案是它所依赖的哲学。 Netty是设计来给你在API和实现上最佳体验的。它不是有迹可循的东西，但你在阅读这片指南将察觉这一哲学会让你玩Netty的生活变得更好过。

### 开始

这篇指南围绕着Netty的核心结构举出了几个例子来让你迅速上手。当你看完这篇文章后将能够立刻写出一个基于Netty的客户端和服务端。

如果你更喜欢自顶而下的学习方法，可以从第二篇开始，架构概览，然后返回到这里。

#### 开始之前

本篇文章所介绍的示例程序所需要的最小运行需求只有两点：最新版本的Netty和JDK 1.6及以上。最新版本的Netty可以在[项目下载页](http://netty.io/downloads.html)找到。

当你读的时候，可能对本篇介绍的这些类有很多疑问。当你想要深入了解这些时，请查询API手册。本篇文章里的所有类都会关联到在线API手册以便于你阅读。同时，如果你发现了有不对的信息，语法错误，排版错误或者你有提高这片文档的好主意，请务必果断[联系Netty项目社区](http://netty.io/community.html) 。

#### 写一个Discard服务器

在Netty的世界里，最简单的协议不是"Hello World!"，而是[DISCARD](http://tools.ietf.org/html/rfc863)。这个协议将丢弃任何接收到的数据，不做任何响应。

实现`DISCARD`协议要做的事情就是忽略所有接收到的数据。让我们直接从处理实现开始，它负责处理Netty产生的I/O事件。

```java
package io.netty.example.discard;
import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHanlderAdapter;
/**
 * 处理服务端通道
 */
public class DiscardServerHandler extends ChannelInboundHanderAdapter{ //(1)
  @Override
  public void channelRead(ChannelHandlerContext ctx,Object msg){ //(2)
    //默默的丢弃接收到的数据
    ((ByteBuf) msg).release(); //(3)
  }
  @Override
  public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause){ //(4)
    //出现异常时关闭链接
    cause.printStackTrace();
    ctx.close();
  }
}
```

1. `DiscardServerHandler`继承了 [`ChannelInboundHandlerAdapter`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html)，它是[`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html)的一个实现。`ChannelInboundHandler`提供了各种事件处理方法供你重写。目前，它只需要继承`ChannelInboundHandlerAdapter`而不用自己实现处理接口。

2. 我们重写了`channelRead()`事件处理方法。这个方法将在接收到信息时被调用，无论何时从客户端接收到新的数据。在这个例子中，接收的信息的数据类型是[`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)。

3. 为了实现`DISCARD`协议，处理器需忽略接收到的信息。`ByteBuf`是一个引用计数对象，它需要通过`release()`方法进行显示的释放。请牢记，释放一切传递给处理器的引用计数对象是处理器应尽的责任。通常，`channelRead()`处理方法像如下这样实现：

   ```java
   @Override
   public void channelRead(ChannelHandlerContext ctx,Object msg){
    try{
      //对msg做些什么
    } finally{
      ReferenceCountUtil.release(msg);
    }
   }
   ```

4. 当Netty因为I/O错误或处理器实现在进行事件处理中抛出异常时，将会调用`exceptionCaught()`方法事件处理。虽然这个方法的实现会根据你对于异常情况下的处理而有所不同，但在大多数情况下，对被获取的异常应该记录日志，关闭与之相关的通道。例如，你可能想要在关闭连接之前发送一个包含错误码的响应信息。


目前为止一切正常。我们已经实现了`DISCARD`服务的一半。剩下的是写一个`main()`方法来调用`DiscardServerHandler`来启动服务。

```java
package io.netty.example.discard;
import io.netty.bootstrap.ServerBootStrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelIntializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
/**
 * 忽略一切进来的数据
 */
public class DiscardServer{
  private int port;
  public DiscardServer(int port){
    this.port=port;
  }
  public void run() throws Exception{
    EventLoopGroup bossGroup=new NioEventLoopGroup(); //(1)
    EventLoopGroup workerGroup=new NioEventLoopGroup();
    try{
      ServerBootstrap b=new ServerBootstrap(); //(2)
      b.group(bossGroup,workerGroup)
        .channel(NioServerSocketChannel.class) //(3)
        .childHanlder(new ChannelInitializer<SocketChannel>(){ //(4)
          @Override
          public void initChannel(SocketChannel ch) throws Exception{
            ch.pipeline().addLast(new DiscardServerHandler());
          }
        })
        .option(CHannelOption.SO_BACKLOG,128) //(5)
        .childOption(ChannelOption.SO_KEEPALIVE,true); //(6)
      //绑定并开始接收输入的连接
      ChannelFuture f=b.bind(port).sync(); //(7)
      //在服务器套接字关闭之前保持等待
      //在这个例子中，这不会发生，但你可以优雅的做这件事
      //关闭服务
      f.channel().closeFuture().sync();
    } finally{
      workerGroup.shutdownGracefully();
      bossGroup.shutdownGracefully();
    }
  }
  public static void main(String[] args) throws Exception{
    int port;
    if(args.length>0){
      port=Integer.parseInt(args[0]);
    }else{
      port=8080;
    }
    new DiscardServer(port).run();
  }
}
```

1. [`NioEventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html)是一个处理I/O操作的多线程事件循环。Netty为不同类型的传输提供了多种多样的[`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 的实现。在这个例子中我们要实现一个服务端应用，因此将用到两个[`NioEventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html)。第一个，通常称为‘工头’，接受一个输入连接。第二个，通常称为‘工人’，当工头接受连接并将接收到的连接分配给工人时，工人将处理接收到的连接的流量。有多少线程被使用以及如何匹配到创建的通道([`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html))取决于[`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 的实现以及可能甚至通过构造器进行配置。
2. [`ServerBootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html)是一个用来启动服务器的辅助类。你可以使用通道来直接启动服务。然而需要知道的是，这是一个乏味的过程，在大多数情况下你并不需要做这些。
3. 在这里，我们指定使用[`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) 类，该类用于实例化新的通道(Channel)以接受输入连接。
4. 这里被指定的处理器通常由新接收的通道评定。[`ChannelInitializer`](http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html)是一个特殊的处理器，目的是帮助使用者配置一个新的通道。你很可能想要为新通道配置一个[`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) 添加一些处理器如`DiscardServerHandler`来实现你的网络应用。当应用变得复杂时，你可能会在管道上添加更多的处理器，最终将这个匿名类升级为一个顶级类。
5. 你也可以设置指派给通道实现的参数。我们在写一个TCP/IP服务，所以我们被允许设置套接字选项如`tcpNoDelay`和`keepAlive`。请查阅接口文档 [`ChannelOption`](http://netty.io/4.0/api/io/netty/channel/ChannelOption.html) 以及相应的[`ChannelConfig`](http://netty.io/4.0/api/io/netty/channel/ChannelConfig.html)实现，来对支持的`ChannelOption`有个宏观的概念。
6. 你注意到`option()`和`childOption()`了么？`option()`是为了接受输入连接的[`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) 。`childOption()`是为了父[`ServerChannel`](http://netty.io/4.0/api/io/netty/channel/ServerChannel.html)所接受的通道，在本例中是 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) 。
7. 我们已经整装待发了。剩下的就是绑定端口并启动服务。在这里我们绑定端口`8080`到机器上所有NIC（网络接口卡，network interface card）。你可以使用不同的绑定地址调用`bind()`方法多次。

祝贺你，你刚刚完成了基于Netty的第一个服务。

#### 查看接收的数据

现在我们已经写了自己的第一个服务器，我们需要测试一下它是否真的能工作。最简单的方式就是使用*telnet*命令。例如，你可以在控制台命令行键入`telnet localhost 8080`，然后输入一些东西。

然而，我们能说这个服务器正常工作了么？我们无法确定，因为他是一个丢包服务。你不会得到任何响应。为了证明它确实工作了，让我们来修改一下服务，打印它所接收到的东西。

我们已经知道`channelRead()`方法会在任意数据接收是激活。让我们在`DiscardServerHandler`的`channelRead()`里放入一些代码：

```java
@Override
public void channelRead(ChannelHandlerContext ctx,Object msg){
  ByteBuf in = (ByteBuf) msg;
  try{
    while(in.isReadable()){ //(1)
      System.out.print((char) in.readByte());
      System.out.flush();
    }
  } finally{
    ReferenceCountUtil.release(msg); //(2)
  }
}
```

1. 这个效率低下的循环其实可以简化为：`System.out.pritnln(in.toString(io.netty.util.CharsetUtil.US_ASCII))`
2. 这里可以做`in.release()`，作为附加选项。

如果你再度运行*telnet*命令，你会发现服务器会打印出接收到的内容。

丢包服务器的全部代码位于发布包中的[`io.netty.example.discard`](http://netty.io/4.0/xref/io/netty/example/discard/package-summary.html)。

#### 编写一个响应服务器

到目前为止，我们一直在消费数据而不做任何响应。然而，一个服务器通常需要对一个请求作出响应。现在让我们来学习一下如何通过实现一个 [`ECHO`](http://tools.ietf.org/html/rfc862)协议来向客户端响应信息，返回任何接收到的数据。

与先前的丢包服务器唯一不同之处在于，它将接收到的数据返回，而不是仅仅在控制台打印输出。因此，只需要修改一下`channelRead()`方法就足够了：

```java
@Override
public void channelRead(ChannelHandlerContext ctx,Object msg){
  ctx.write(msg); //(1)
  ctx.flush(); //(2)
}
```

1. 一个[`ChannelHandlerContext`](http://netty.io/4.0/api/io/netty/channel/ChannelHandlerContext.html)对象提供了多种操作，使得你能够出发不同的I/O事件和操作。在这里，我们触发`write(Object)`来逐字输出接收到的信息。请注意我们没有像在`DISCARD`例子中那样释放接收的信息。因为在写出到线上时Netty将为你释放它。
2. `ctx.write(Object)`并没有将信息写出到线上。它在内部是缓冲起来，而后通过`ctx.flush()`来冲出。可以通过调用`ctx.writeAndFlush(msg)`来简化。

当你再次运行*telnet*命令，你将看到服务器将返回一切你发送给它的内容。

响应服务器的所有代码位于发布的 [`io.netty.example.echo`](http://netty.io/4.0/xref/io/netty/example/echo/package-summary.html) 包内。