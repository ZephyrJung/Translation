# Netty 4.x用户手册

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

#### 编写一个时间服务器

这一节要实现的协议是[`TIME`](http://tools.ietf.org/html/rfc868)。这与前面的例子不同。它发送了一个包含32位整数的信息，且不接收任何请求，并在信息发送完成后关闭连接。在这个例子里，你将学会如何组织并发送一个信息，然后在完成时关闭连接。

由于我们打算忽略任何接收的数据，并且在连接创建时立刻发送一个信息，这次我们不能使用`channelRead()`方法。取而代之，我们需要重写`channelActive()`方法。实现如下：

```java
package io.netty.example.time;
public class TimeServerHandler extends ChannelInboundHandlerAdapter{
  @Override
  public void channelActive(final ChannelHandlerContext ctx){ //(1)
    final ByteBuf time = ctx.alloc().buffer(4); //(2)
    time.writeInt((int)(System.currentTimeMillis()/1000L+2208988800L));
    final ChannelFuture f=ctx.writeAndFlush(time); //(3)
    f.addListener(new ChannelFutureListener(){
      @Override
      public void operationComplete(ChannelFuture future){
        assert f==future;
        ctx.close();
      }
    }); //(4)
  }
  @Override
  public void exceptionCaught(ChannelHandlerContext ctx,Throwable cause){
    cause.printStackTrace();
    ctx.close();
  }
}
```

1. 不言而喻，`channelActive()`方法将在连接创建时被激活，准备产生流量。让我们在这个方法中写入一个32位整数来代表当前时间。

2. 为了发送新的信息，我们需要分配一个新的缓冲区来保存。32位整数需要一个容量为4个字节的[`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)，通过`ChannelHandlerContext.alloc()`获取当前的[`ByteBufAllocator`](http://netty.io/4.0/api/io/netty/buffer/ByteBufAllocator.html)并分配一个新的缓冲区。

3. 像往常一样，我们来写信息的构造。

   但是请等一下，flip在哪儿？难道在NIO中，发送信息之前不应该调用`java.nio.ByteBuffer.flip()`么？`ByteBuf`并不包含这个方法，因为它有两个指针。一个指向读操作，另一个是写操作。当你向`ByteBuf`写什么东西的时候，写指针地址索引将会增长，读指针不变。读写地址索引分别代表了信息的开始与结束。

   相反，如果没有flip方法，NIO缓冲区并没有提供一个简便的方式来辨别信息内容的开始和结束。如果你忘记flip缓冲区将会陷入麻烦，因为空或者错误的数据将被发送。这样的错误不会在Netty中发生，因为对于不同的操作类型，我们有不同的指针。当你习惯与此你会发现这让你的日子更好过了——没有flip out的日子!

   另一个需要知道的点是`ChannelHandlerContext.write()`（以及`writeAndFlush()`）方法返回一个 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html)。一个 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html)代表一个尚未发生的I/O操作。这意味着，任何请求操作可能都还没有执行，因为Netty中所有操作都是异步的。例如，下面这段代码可能会在信息发送前就关闭了连接：

   ```java
   Channel ch=...;
   ch.writeAndFlush(message);
   ch.close();
   ```

   因此，你需要在`write()`返回的[`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html)完成以后调用`close()`方法。它将会在写操作完成后通知监听者。请注意，`close()`也可能不会立刻关闭连接，并且它返回一个[`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html)。

4. 那当请求完成后，我们如何获得通知呢？很简单，只需要在返回的`ChannelFuture`上添加一个 [`ChannelFutureListener`](http://netty.io/4.0/api/io/netty/channel/ChannelFutureListener.html)。这里，我们创建了一个新的异步 [`ChannelFutureListener`](http://netty.io/4.0/api/io/netty/channel/ChannelFutureListener.html)来在操作结束时关闭通道。

   另一种选择，是使用预定义的监听器来简化代码：

   ```java
   f.addListener(ChannelFutureListener.CLOSE);
   ```

为了测试我们的时间服务器像预期的那样工作，可以使用UNIX的`rdate`命令：

```shell
$ rdate -o <port> -p <host>
```

`<port>`是你在`main()`方法中定义的端口号，`<host>`一般是`localhost`。

#### 编写一个时间客户端

与`DISCARD`和`ECHO`服务器不同，我们需要为`TIME`协议准备一个客户端，因为人无法手工将32位二进制数据转换为日历上的一个日期。在这一节，我们讨论如何确保服务端正确的工作，并学习如何用Netty写一个客户端。

Netty关于服务端和客户端最大和唯一的区别在于使用了不同的[`Bootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/Bootstrap.html) 和[`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html)实现。请看一下下面的代码：

```java
package io.netty.example.time;
public class TimeClient{
  public static void main(String[] args) throws Exception{
    String host=args[0];
    int port=Integer.parseInt(args[1]);
    EventLoopGroup workerGroup=new NioEventLoopGroup();
    try{
      Bootstrap b=new Bootstrap(); //(1)
      b.group(workerGroup); //(2)
      b.channel(NioSocketChannel.class); //(3)
      b.option(ChannelOption.SO_KEEPALIVE,true); //(4)
      b.handler(new ChannelInitializer<SocketChannel>(){
        @Override
        public void initChannel(SocketChannel ch) throws Exception{
          ch.pipeline().addLast(new TimeClientHandler());
        }
      });
      //启动客户端
      ChannelFuture f=b.connect(host,port).sync(); //(5)
      //在连接关闭之前保持等待
      f.channel().closeFuture().sync();
    }finally{
      workerGroup.shutdownGracefully();
    }
  }
}
```

1. [`Bootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/Bootstrap.html)与 [`ServerBootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html) 类似，只不过它用于非服务端通道，如客户端或无连接通道。
2. 如果你只指定了一个 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html)，它将既做工头也做工人。尽管工头并不用于客户端。
3. [`NioSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioSocketChannel.html) 用来创建一个客户端的通道，而不是 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)。
4. 注意这里我们没有使用`childOption()`，因为客户端的 [`SocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/SocketChannel.html) 没有父类。
5. 我们应当调用`connect()`方法而非`bind()`方法。

如你所见，这与服务端代码不尽相同。那么[`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 实现又怎样呢？他应该从服务器接收32位整数，并翻译成人工可读格式，打印翻译的时间，关闭连接：

```java
package io.netty.example.time;
import java.util.Date;
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. 在TCP/IP中，Netty从对等端读取数据到一个 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)。

这看起来很简单，与服务端的示例没有什么大不同。然而，这个处理器有时候会拒绝工作并发出一个`IndexOutOfBoundsExcepiton`。在下一节我们来讨论为什么这个会发生。

#### 处理基于流的传输

##### 关于套接字缓冲区的一个小注意事项

在基于流传输如TCP/IP中，接收的数据存储在一个套接字缓冲区中。不幸的是，这个流传输缓冲区并非是一个包的队列，而是一个字节队列。意思是说，即便你发送了两个独立包的信息，操作系统并不会把它们当做两条信息，而是当做一堆字节。因此无法保证你所读的内容就是远程另一端所写的内容。例如，假定一个操作系统的TCP/IP栈接收到了三个包：

`ABC` `DEF` `GHI` 

由于基于流协议的一般属性，在你的应用里有很大概率读取为如下的片段形式：

`AB` `CDEFG` `H` `I`

因此，在接收的部分，不论是服务端还是客户端，都应该将接收到的数据分割成一个或者多个有意义的帧，以便能够被应用逻辑方便的理解。如上面这个例子，接收的数据应该封装成为下面这样的帧：

`ABC` `DEF` `GHI`

##### 第一个方案

现在，让我们返回`TIME`客户端这个例子。这里有同样的问题。一个32位整数是一个很小量的数据，并且它不可能经常被碎片化。然而，问题是它可以被碎片化，并且随着流量的提高，碎片化的可能性也会提高。

最简单的办法是创建一个内部的累积缓冲区，等待所有4字节被放进来。如下是一个修改后的`TimeClientHandler`，解决了这个问题：

```java
package io.netty.example.time;
import java.util.Date;
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
    private ByteBuf buf;
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. 一个 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 有两个生命周期监听器方法：`handlerAdded()`和`handlerRemoved()`。你可以执行任意一个初始化（反初始化）任务，只要它没有长时间阻塞。
2. 首先，所有接收的数据都应该积累到`buf`中。
3. 然后，这个处理器必须检查`buf`是否包含了足够的数据，这个例子里是4个字节，然后再进行真正的业务逻辑。当有更多数据到达时，Netty将会再次调用`channelRead()`方法，最终将累积所有4个字节。

##### 第二个方案

虽然第一个方案解决了`TIME`客户端的问题，但是修改后的处理器看起来不甚简洁。想象一个更复杂的协议，融合了如变量长度的多个字段。你的 [`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html) 实现将很快变得无法维护。

你可能已经注意到了，你可以在[`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html)上添加不止一个 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) ，因此，你可以将单个的 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 划分成多个模块，来降低你应用的复杂度。例如，你可以将`TimeClientHandler`划分成两个处理器：

- `TimeDecoder`用来处理碎片问题，以及
- 初始化`TimeClientHandler`的简单版本

幸运的是，Netty提供了一个可扩展的类来帮你写出第一个东西：

```java
package io.netty.example.time;
public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        out.add(in.readBytes(4)); // (4)
    }
}
```

1. [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 是 [`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html) 的一个实现，它使得处理碎片问题变得更容易了。
2. [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 调用了`decode()`方法，这个方法内部维护了一个累积缓冲区来接收任何新来的数据。
3. `decode()`能够决定在没有累积缓冲区足够的数据时不向`out`添加内容。 [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 将会在更多数据接收时再次调用`decode()`。
4. 如果`decode()`向`out`添加了一个对象，这意味着解码器成功的解码了一个信息。 [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 将会放弃读取部分累计缓冲区。请记住你没有必要解码多个信息。[`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 将会持续调用`decode()`方法直到没有可向`out`添加的内容为止。

现在我们有另一个处理器添加到 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) 中，我们需要修改`TimeClient`中的 [`ChannelInitializer`](http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html) 实现：

```java
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果你乐于冒险，可以尝试使用 [`ReplayingDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ReplayingDecoder.html) 让解码器进一步简化。当然，你需要查询API手册来获取更多信息。

```java
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
      	ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```

另外，Netty提供了开箱即用的解码器，可以让你很容易的实现大多数协议，帮你避免以单个无法维护的处理器实现告终。请查看下面的包中的例子来获取更多细节：

- [`io.netty.example.factorial`](http://netty.io/4.0/xref/io/netty/example/factorial/package-summary.html) 二进制协议
- [`io.netty.example.telnet`](http://netty.io/4.0/xref/io/netty/example/telnet/package-summary.html) 文本行协议

#### 用POJO说话而非`ByteBuf`

目前我们所看到的代码都是用 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 作为协议信息的主要数据结构。在这一节，我们将升级`TIME`协议的客户端和服务端例子，使用POJO而不是 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)。

在 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 中使用POJO的有点显而易见。你的处理器将更容易维护，即从`ByteBuf`抽取的信息能够被分离的代码重用。在`TIME`客户端和服务端的例子中，我们只读取了32位整数，而直接使用`ByteBuf`并不是主要问题。然而，当你实现一个现实世界中的协议时，你会发现做分离是很有必要的。

首先，让我们定义一个新的类型，名为`UnixTime`。

```java
package io.netty.example.time;
import java.util.Date;
public class UnixTime {
    private final long value;
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    public UnixTime(long value) {
        this.value = value;
    }
    public long value() {
        return value;
    }
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}
```

现在我们可以修改`TimeDecoder`来产生一个`UnixTime`而非 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)。

```java
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }
    out.add(new UnixTime(in.readUnsignedInt()));
}
```

使用了新的解码器，`TimeClientHandler`将不再使用 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) ：

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

更简单优雅了，对么？同样的技术将应用在服务端。这次让我们更新一下`TimeServerHandler`：

```java
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```

现在，唯一缺少的部分就是编码器了，它是 [`ChannelOutboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelOutboundHandler.html) 的一个实现，将`UnixTime`翻译回一个 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 。这比写一个解码器容易得多，因为在对信息编码时不需要处理数据包分片和组装。

```java
package io.netty.example.time;
public class TimeEncoder extends ChannelOutboundHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int)m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

1. 这一行有几个重要的地方：

   首先，我们传递了一个原生态 [`ChannelPromise`](http://netty.io/4.0/api/io/netty/channel/ChannelPromise.html) ，当编码后的数据实际向线上写出时，Netty标记它为成功或者失败。

   其次，我们不调用`ctx.flush()`。这里有一个处理器方法`void flush(ChannelHandlerContext ctx)`，它的目的就是重写`flush()`操作。

为了更进一步简化，你可以使用 [`MessageToByteEncoder`](http://netty.io/4.0/api/io/netty/handler/codec/MessageToByteEncoder.html)：

```java
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}
```

最后一个剩下的任务是将`TimeEncoder`在`TimeServerHandler`之前插入到服务端的 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) ，这作为一个练习任务略过。

#### 关闭应用

关闭一个Netty应用通常和使用`shutdownGracefully()`关闭所有你创建的[`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html)一样简单。它返回一个 [`Future`](http://netty.io/4.0/api/io/netty/util/concurrent/Future.html) 来告诉你 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 何时完全终止，以及所有属于这个组的[`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html)都已经关闭。

#### 总结

在本篇文章中，我们使用一个关于如何基于Netty写一个完全工作的网络应用的展示进行了一次快速的Netty之旅。

即将到来的篇章中将包含关于Netty的更多章节，我们鼓励你复读 [`io.netty.example`](https://github.com/netty/netty/tree/4.0/example/src/main/java/io/netty/example) 包下的Netty示例。

同时请知道 [the community](http://netty.io/community.html) 永远在等待你的问题和反馈来帮助提高Netty及其文档。