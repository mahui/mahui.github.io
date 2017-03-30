---
title: Netty用户指南4.x
date: 2017-03-27 15:06:43
categories:
- 文档
tags:
- Netty
- Java
- 翻译
---

> Netty 用户指南中文版  
> Based on [User guide for 4.x](http://netty.io/wiki/user-guide-for-4.x.html) .

# 序言 [Preface](http://netty.io/wiki/user-guide-for-4.x.html#preface)  

## 问题 [The Problem](http://netty.io/wiki/user-guide-for-4.x.html#the-problem)

如今我们使用通用的应用程序或者库来彼此通讯。比如，我们经常使用 HTTP 客户端库从远程服务器接收信息或者执行远程服务的调用。但无论如何，一个通用的协议或者它们的实现有时候可用性不是太好。这就是为什么有时候我们并不使用通用的 HTTP 协议来交换大数据、电子邮件信息以及金融数据和多人网络游戏数据这种近实时的数据。所需要的是为了一个专门的目的去实现一个高度优化的协议。比如，你可能需为了基于 AJAX 的聊天应用，媒体传输，或者大文件传输去实现一个优化的 HTTP 服务。你甚至可以为了你的需求去精确的定义并实现一个全新的协议。另外一个经常发生的情况是你必须去处理系统中遗留的专门协议去确保与旧系统的交互。这种情况下重要的是，我们如何能够快速的实现该协议，并且不会牺牲应用程序的稳定性和性能。

## 解决方案 [The Solution](http://netty.io/wiki/user-guide-for-4.x.html#the-solution)

[Netty](http://netty.io) 是一个提供了异步事件驱动 (asynchronous event-driven) 网络应用程序框架和工具的杰作，可以用来快速开发可维护的，高性能 · 高可用的服务端和客户端协议。

从另一方面来讲，Netty 是一个可以快速开发网络程序，如客户端和服务端协议的 NIO 客户端/服务端框架。它大大的简化了如 TCP 和 UDP 这类套接字服务的网络应用程序的开发并优化了其开发流程。

“快速简单”并不意味着开发出的应用程序的可维护性变差或者遇到性能问题。 Netty 从大量的如 FTP、SMPT、HTTP 以及各种二进制文本传输协议的实现中获得经验，并精心设计。因此， Netty 成功的获得了可以快速开发，并且不妥协性能、稳定性以及灵活性的方法。

一些用户用户或者发现过其他一些网络应用框架也声称有以上优势，可能会想问 Netty 和它们有何不同。答案是，设计哲学。 Netty 从一开始就设计得为你提供舒适的 API 及实现。这虽非什么可感知的事物，但你会意识到这些设计哲学让你的生活变得更简单，如同你开始阅读这份指南，并且开始轻松使用 Netty 。

<!-- more -->

# 开始 [Getting Started](http://netty.io/wiki/user-guide-for-4.x.html#getting-started)

本章指南围绕着 Netty 的核心结构，据一些简单的例子，让你轻松开始。本章结束后，你将可在 Netty 之上编写客户端和服务端。

如果你学习东西的时候喜欢使用自上而下分析法，那么你也许可以从第二章，架构总览开始，然后回到这里。

## 开始之前 [Before Getting Started](https://github.com/netty/netty/wiki/User-guide-for-4.x#before-getting-started)

要想运行本章中所引入的例子，需要两个最小的依赖：Netty 最新的稳定版 和 JDK 1.6 或者更新。Netty 最新的稳定版可以从 [项目下载页面](http://netty.io/downloads.html) 下载。可以从 JDK 供应商的网站下载正确版本的 JDK 。

在你阅读的过程中，你可能对本章中引入的类有一些疑问。如果你想知道它们可以参考 API 文档。为了你的方便，文档中所有的类名都是引用的在线 API 文档。当然了，如果本文档中有任何错误信息，以及语法和打印中的错误，或者你有什么完善本文档的好的建议，可以[联系Netty项目社区](http://netty.io/community.html)。

## 写一个 Discard 服务 [Writing a Discard Server](https://github.com/netty/netty/wiki/User-guide-for-4.x#writing-a-discard-server)

世界上最简单的协议是 [`DISCARD`](http://tools.ietf.org/html/rfc863) ，而不是 "Hello,World!"。该协议丢弃任何收到的数据而不做任何回应。
要实现 `DISCARD` 协议，你唯一要做的事情就是忽略任何收到的消息。让我们直接从一个 handler 的实现开始，handler 是 Netty 生成的用来处理 I/O 事件的。

```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * 处理一个服务端的 channel。
 */
public class DiscardServerHandler extends ChannelInboundHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // 默默的丢弃收到的数据。
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // 当引发异常时关闭连接。
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. `DiscardServerHandler` 继承自 [`ChannelInboundHandlerAdapter`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html)，该类实现了 [`ChannelInboundHandler` ](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html).[`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html) 接口，提供了多种事件处理方法，你可以重写这些方法。现在，你只需要继承 [`ChannelInboundHandlerAdapter`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandlerAdapter.html) ,而无需自己实现接口中的处理方法。

2. 我们重写了 `channelRead()` 方法。无论何时从客户端收到消息时，该方法被调用。在这个例子中，收到的消息类型是 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html)

3. 为了实现 `DISCARD` 协议，该处理器必须忽略收到的消息。  [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 是一个引用计数对象，所以必须显式的调用 `release()` 方法来释放资源。请谨记，任何传递到处理器引用计数对象，处理器都有责任将其释放掉。通常， `channelRead()` 方法的实现如下：
```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```
4. 当 Netty 犹豫 I/O 错误或者处理器处理事件时抛出异常，导致 Throwable 对象出现时，`exceptionCaught() ` 方法被调用。大多数情况下，补货到的异常应该被记录，与其相关的 channle 也应该在这儿被关闭，当然了，该方法的实现也会因你的不同异常处理方案而不同。比如，你也许想在关闭连接之前发送一个包含错误代码的响应消息。

目前来说一切都很好，我们实现了一半的 `DISCARD` 服务。现在剩下我们去在 `DiscardServerHandler` 中写一个 `main()` 方法来启动服务。


```java
package io.netty.example.discard;

import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * 丢弃任何进来的数据。
 */
public class DiscardServer {

    private int port;

    public DiscardServer(int port) {
        this.port = port;
    }

    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)

            // 绑定端口并且开始服务接收进来的连接。
            ChannelFuture f = b.bind(port).sync(); // (7)

            // 等待 server socket 关闭。
            // 在这个例子中，这不会发生，但你可以这样优雅的关闭你的服务。
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

1. [`NioEventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html) 是一个用来处理 I/O 操作的多线程事件循环器。 Netty 为不同类型传输提供了各种各样的 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 的实现。这个例子我们实现了一个服务端的应用，声明了两个 [`NioEventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/nio/NioEventLoopGroup.html) 实例。第一个，通常称为 "boss",用来接收进来的连接。第二个，通常称为 "worker", 用来处理接收到连接，一旦 boss 接收到连接，就将其注册到 worker 上。使用多少个线程，如何将其映射到已经创建好的 [`channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html)S 依赖于 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 的实现，也可以通过构造方法进行配置。

2. [`ServerBootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html) 是一个建立 server 的辅助类。你也可以通过 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 直接建立一个 server。但请注意，这是一个非常冗长的过程，很多时候，你并不需要这么做。

3. 在这儿，我们指定使用 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) 类来实例化一个新的 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 来接收进来的连接。

4. 在这儿指定的处理器通常由新接收的 Channel 进行评估。[`ChannelInitializer`](http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html) 是一个特殊的处理器，用来帮助用户配置一个新的 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html)。很可能你会想通过添加处理器,如 `DiscardServerHandler` ,来配置新 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 的 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) ,用来实现你的网络应用。随着应用变得复杂，你会往管道添加更多的处理器，最终你可能会把这个匿名类提取为一个单独的类。

5. 你可以设置指定 Channel 实现的特定参数。我们在写一个 TCP/IP 服务，所以我们可以设置一些 socket 的选项，比如 `tcpNoDelay` 和 `keepAlive` 。请参照 [`ChannelOption`](http://netty.io/4.0/api/io/netty/channel/ChannelOption.html) API 文档以及特定的 [`ChannelConfig`](http://netty.io/4.0/api/io/netty/channel/ChannelConfig.html) 实现，以获得有关 `ChannelOptionS` 支持的概述。

6. 你有注意到 `option()` 和 `childOption` 方法了吗？ `option()` 是为 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) 用来接收进来的连接。 `childOption()` 是为被父通道 [`ServerChannel`](http://netty.io/4.0/api/io/netty/channel/ServerChannel.html) 接收到的 `ChannelS` ，在本例中指的是 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)。

7. 我们现在已经准备好了。剩下的就是绑定端口和启动服务了。在这儿，我们绑定了机器所有网卡的 `8080` 端口。你现在可以调用多次 `bind()` 方法（用不同的绑定地址）。

恭喜你！你已经基于 Netty 完成里你的第一个服务端程序。

## 查看接收到的数据[Looking into the Received Data](http://netty.io/wiki/user-guide-for-4.x.html#looking-into-the-received-data)

现在我们已经写好了我们的第一个服务端程序，我们需要测试它是否正常运行。最简单的方法是使用 `telnet` 命令行。比如，你可以在终端输入 `telnet localhost 8080` 命令行，然后输入一些东西。

但这样就能表示我们的服务运行正常吗？我们实际上并不能准确知道，因为它是个 `discard` 服务。它并不会返回任何信息。为了证明它是真的正常运行，我们可以修改一下服务端程序让它把收到的数据打印出来。

我们已经知道当数据被接收到的时候会调用 `channelRead()` 方法。可以在 `DiscardServerHandler` 的 `channelRead()` 方法中加一些代码：
```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}
```

1. 这个低效的循环实际上可以替换为: `System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))`
2. 或者，你可以在这儿调 `in.release()`

如果你再次调用 `telnet` 命令，你会看到服务端程序会把收到的数据打印出来。

discard 服务的全部代码放在 [`io.netty.example.discard`](http://netty.io/4.0/xref/io/netty/example/discard/package-summary.html) 包中。

## 写一个 Echo 服务 [Writing an Echo Server](http://netty.io/wiki/user-guide-for-4.x.html#writing-an-echo-server)

目前为止，我们的服务都是消费数据，而不做任何响应。然而作为一个服务，往往是需要对请求做出响应的。让我们学习如何通过实现 [`ECHO`](http://tools.ietf.org/html/rfc862) 协议（如论接收到任何数据都原样返回）来给客户端返回响应数据。

与我们先前小节中实现的 discard 服务唯一不同的是，我们将接收到的数据发出，而不是将其打印到控制台。因此，需要再次改动 `channelRead()` 方法：

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
   ctx.write(msg); // (1)
   ctx.flush(); // (2)
}
```

1. [`ChannelHandlerContext`](http://netty.io/4.0/api/io/netty/channel/ChannelHandlerContext.html) 对象提供了各种操作可以用来触发各种 I/O 事件以及操作。在这儿我们调用 `write(Object)` 方法将接收到的消息一字不差的返回。请注意，在这儿我们没有像 `DISCARD` 服务一样 release 接收到的消息，因为 Netty 会在你将其写出至网络时将其 release。

2. `ctx.write(Object)` 并不会将消息写到网络上。它只是被放在内部缓冲区，然后被 `ctx.flush()` 方法刷新至网络上。或者你可以用更简洁的 `ctx.writeAndFlush(msg)` 方法。

如果你再次运行 `telnet` 命令，你会发现无论你发送什么内容到服务器上，它都会将其原样返回。

echo 服务的全部代码可以在 [`io.netty.example.echo`](http://netty.io/4.0/xref/io/netty/example/echo/package-summary.html) 包中找到。

## 写一个时间服务 [Writing a Time Server](http://netty.io/wiki/user-guide-for-4.x.html#writing-a-time-server)

本小节需要实现的协议是 [`TIME`](http://tools.ietf.org/html/rfc868) 协议。与之前例子不同的是，它不会接收任何请求，会发送一条消息，里面包含了一个 32 位数字，当消息发送之后，会关闭连接。在本例中，你将了解到如何构造并发送一条消息，并且在服务结束的时候关闭连接。

因为你不打算接受任何消息，而连接一旦建立，你就会将消息发出，所以我们这次不能使用 `channelRead()` 方法。我们需要重写 `channelActive()` 方法。以下为实现：

```java
package io.netty.example.time;

public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));

        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. 解释一下， `channelActive()` 方法会在连接建立好可以准备交换数据时被调用。我们在这儿写一个可以表示当前时间的 32 位数字。

2. 为了发送一条新消息，我们需要分配一个新的缓冲区 (buffer) 用来存放消息。我们将要写一个 32 位的数字，因此，需要一个容量至少为 4 个字节的 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 。通过 `ChannelHandlerContext.alloc()` 方法获取当前的 [`ByteBufAllocator`](http://netty.io/4.0/api/io/netty/buffer/ByteBufAllocator.html) 对象并且分配一个新的缓冲区 (buffer)。

3. 像往常一样，我们写构造的数据。  
但等等， flip 方法呢？ 我们在 NIO 中发送消息之前不需要调用 `java.nio.ByteBuffer.flip()` 方法吗？ `ByteBuf` 没有这个方法，因为它有两个指针: 一个是用来读操作，另一个是用来写操作。当你往 `ByteBuf` 写入数据的时候，写指针(writer index)会变大，而读指针(reader index)没有变化。读指针(reader index) 和 写指针(writer index) 分别体现了消息的开始和结尾位置。  
对比 NIO 缓冲区，其并没有提供一个清晰的方法可以计算出消息的开始和结束位置，除了调用 flip 方法。当你忘记 flip 缓冲区时，你会遇到麻烦，因为有可能发不出数据或者发错数据。而这样的错误不会在 Netty 发生，因为针对不同的操作，我们有不同的指针。你会发现习惯了没有 flipping out 的生活变得更简单！  
另一个需要注意的点是, `ChannelHandlerContext.write()` (以及 `writeAndFlush()`) 方法返回一个 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html) 对象。一个 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html) 对象代表一个 I/O 操作还没发生。这意味着任何请求操作都有可能尚未执行，因为 Netty 中所有的操作都是异步的。比如，如下例子中，有可能在消息发出去之前关闭连接:
    ```java
    Channel ch = ...;
    ch.writeAndFlush(message);
    ch.close();
    ```
    因此你需要在 `write()` 方法返回的 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html) 对象完成之后关闭连接，当写操作完成后，它会通知它的监听程序。请注意，`close()` 方法也有可能不会立即关闭连接，他也会返回一个 [`ChannelFuture`](http://netty.io/4.0/api/io/netty/channel/ChannelFuture.html) 对象。
4. 当写请求完成时我们如何得到通知呢? 很简单，只需要对返回的 `ChannelFuture` 添加 [`ChannelFutureListener`](http://netty.io/4.0/api/io/netty/channel/ChannelFutureListener.html) 即可。在这儿我们创建了一个新的匿名类 [`ChannelFutureListener`](http://netty.io/4.0/api/io/netty/channel/ChannelFutureListener.html) , 当操作完成时它会关闭 `Channel`。  
要不然你也可以使用预定义的监听器来简化你的代码：
    ```java
    f.addListener(ChannelFutureListener.CLOSE);
    ```

可以使用 UNIX 命令 `rdate` 来测试我们的时间服务是否能如预期工作：
```shell
$ rdate -o <port> -p <host>
```

这儿的 `<port>` 是在 `main()` 方法中指定的端口号，`<host>` 通常是 `localhost` 。

## 写一个时间客户端 [Writing a Time Client](https://github.com/netty/netty/wiki/User-guide-for-4.x#writing-a-time-client)

不想 `DISCARD` 和 `ECHO` 服务， `TIME` 服务需要一个客户端。因为普通人很难将 32 为二进制数据翻译为日历上对应的时间。在本小节，我们将讨论如何确保服务正确的运行并且学习如何用 Netty 写一个客户端。

Netty 服务端和客户端最大也是唯一的区别就在于使用不同的 [`bootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/Bootstrap.html) 和 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 实现。请注意观察以下代码：

```java
package io.netty.example.time;

public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });

            // 打开客户端
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // 等待连接关闭
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

1. [`bootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/Bootstrap.html) 与 [`ServerBootstrap`](http://netty.io/4.0/api/io/netty/bootstrap/ServerBootstrap.html) 很类似，除了它使用的是非服务端通道 (non-server channels), 如客户端或者无连接通道。

2. 如果你只指定了一个 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) ，它将会既当作 boss 组又会当作 worker 组。虽然客户端并不使用 boss worker 。

3. [`NioSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioSocketChannel.html) 替换 [`NioServerSocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html) , 用来创建客户端的 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 。

4. 留意我们并没有在这儿像 `ServerBootstrap` 一样使用 `childOption()` 方法，因为客户端的 [`SocketChannel`](http://netty.io/4.0/api/io/netty/channel/socket/SocketChannel.html) 没有父。

5. 与服务端调用 `bind()` 不同，我们应该在客户端调用 `connect()` 方法。

如你所见，客户端的代码并没有与服务端差太多。那么 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 该如何实现呢？它应该从服务端接收到 32 为的整数，将其翻译为人类可读的格式并将其打印出来，然后关闭连接：

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

1. 在 TCP/IP 中，Netty 将发送来的数据读到 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 中。

看上去很简单，并且没有发现什么与服务端例子中不同的地方。然而，这个处理器有时候会因为引发 `IndexOutOfBoundsException` 异常，而拒绝工作。我们将在下一小节讨论这为什么会发生。

## 处理基于流的传输 [Dealing with a Stream-based Transport](http://netty.io/wiki/user-guide-for-4.x.html#dealing-with-a-stream-based-transport)

### 套接字缓冲区的一个小小注意事项 [One Small Caveat of Socket Buffer](http://netty.io/wiki/user-guide-for-4.x.html#one-small-caveat-of-socket-buffer)

在基于流传输中，比如 TCP/IP ， 接收到的数据存放在一个套接字接受缓冲区 (socket receive buffer) 中。可惜的是，基于流的传输的缓冲区是一个字节队列，而不是包队列。这就意味着，即使你发了两个独立的数据包，操作系统也不会那它们当两条消息处理，而只是当作一串数组。因此，这不能保证你读到的数据确实就是你远程写入的数据。举个例子，我们假定操作系统的 TCP/IP 栈接收到三个包：

![](https://camo.githubusercontent.com/24ed1176ecca468dfb2b8b017bb927a8715a16f2/687474703a2f2f756d6c2e6d766e7365617263682e6f72672f676973742f3832653366626530653264346466323833323262)

因为基于流的传输协议的这个属性，应用程序读到的数据很有可能是这样的碎片:

![](https://camo.githubusercontent.com/5b595baf5071bf669f81d08b7554064f4142cc69/687474703a2f2f756d6c2e6d766e7365617263682e6f72672f676973742f6233316330626437626266633639666438326436)

因此，无论是服务端还是客户端，在接受数据的部分，应该将收到的数据碎片整理进一个或者多个有意义的帧中，以便可以被程序逻辑轻松理解。在上面的例子中，接收到的数据应该是这样的帧：

![](https://camo.githubusercontent.com/24ed1176ecca468dfb2b8b017bb927a8715a16f2/687474703a2f2f756d6c2e6d766e7365617263682e6f72672f676973742f3832653366626530653264346466323833323262)

### 第一种解决方案 [The First Solution](http://netty.io/wiki/user-guide-for-4.x.html#the-first-solution)

现在让我们回到 `TIME` 客户单的例子中。在这儿我们有同样的问题。一个 32 位的整数是非常小数据，它经常不太可能被拆分为碎片。但问题是，它是有可能被拆分成碎片的，随着交互次数的增长，这种几率也随之增长。

简单的解决方案是，创建一个内部累计缓冲区，等待 4 字节全部被接收到内部缓冲区。下面的修改过的 `TimeClientHandler` 实现解决了这个问题:

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

1. [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 有两个生命周期的监听方法: `handlerAdded()` 和 `handlerRemoved()` 。只要不长时间阻塞，你可以执行任意的初始化\去初始化方法。

2. 首先，接收到的数据应该累计到缓冲区。

3. 然后，处理器必须检测缓冲区是否有足够的数据，本例中是 4 个字节，之后才会处理实际的程序业务逻辑。否则， Netty 会在有更多数据到来时再次调用 `channelRead()` 方法，直到累计到足够 4 个字节。

### 第二种解决方案 [The Second Solution](http://netty.io/wiki/user-guide-for-4.x.html#the-second-solution)

虽然第一种解决方案解决了 `TIME` 客户端的问题，修改过的处理器看上去并不简洁。想象一下，一个复杂的协议，数据由多个不定长度的字段组成。你的 [`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html) 的实现立马会变得不可维护。

就像你已经注意到的，可以添加多个 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 到一个 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) ，因此，你可以将一个大的 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 拆分为多个模块，以降低应用的复杂度。例如，你可以将 `TimeClientHandler` 拆分为两个处理器:

- `TimeDecoder` 用来处理拆分问题，以及
- `TimeClientHandler` 的原始实现。

幸运的是， Netty 提供了一个可被继承的开箱即用的类:

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

1. [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 是 [`ChannelInboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelInboundHandler.html) 的一个实现类，可以让处理拆分问题变得简单。

2.  当数据被接收的时候 [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)  调用 `decode()` 方法来内部维持缓冲区数据累积。

3. 累积缓冲区数据累积不够多的时候， `decode()` 可以添加任何东西到 out 。接收到更多的数据时，[`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 会再次调用 `decode()` 方法。

4. 如果 `decode()` 添加了个对象到 out, 说明解码器成功的解码了消息。 [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 会将累积缓冲区中已经读过的数据丢弃。记住，你不需要解码多消息， [`ByteToMessageDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ByteToMessageDecoder.html) 会反复调用 `decode()` 方法，直到它不添加任何内容到 out。

现在我们有另外一个处理器要添加到 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) , 我们需要修改 `TimeClient` 中 [`ChannelInitializer`](http://netty.io/4.0/api/io/netty/channel/ChannelInitializer.html) 的实现：

```java
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果你是一个大胆到用户，可以尝试下更简单的解码器，[`ReplayingDecoder`](http://netty.io/4.0/api/io/netty/handler/codec/ReplayingDecoder.html) 。当然了，你需要参考下 API 文档以获取更多信息。

```java
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}
```

另外，Netty 提供了很多开箱即用的解码器供你可以很简答的就能实现大多数协议，避免你需要实现一个庞大的不可维护的处理器。请参考下面的包中的更详细的例子：

- [`io.netty.example.factorial`](http://netty.io/4.0/xref/io/netty/example/factorial/package-summary.html) , 二进制协议，以及
- [`io.netty.example.telnet`](http://netty.io/4.0/xref/io/netty/example/telnet/package-summary.html) , 基于文本行的协议。

## 使用 POJO 替代 ByteBuf [Speaking in POJO instead of ByteBuf](http://netty.io/wiki/user-guide-for-4.x.html#speaking-in-pojo-instead-of-bytebuf)

回顾目前所有的例子，都是用 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 作为协议中消息的主要数据结构。在本小节，我们将改善 `TIME` 协议的客户端和服务端，使用 POJO 替换 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 。

在 [`ChannelHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelHandler.html) 中使用 POJO 的优点是显而易见的；将从 ByteBuf 中提取信息部分的代码从处理器中分离出来，可以让处理器变得更可维护，可复用。在 `TIME` 客户端和服务端的例子中，我们只是读取一个 32 位的整数，直接用 ByteBuf 不会是个大问题。然而，你会发现，在现实中的协议，将其分离开来是很有必要的。

首先，让我们定义一个新的对象，UnixTime。

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

我们修改 `TimeDecoder`，生产一个 `UnixTime`，替代 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 。

```java
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```

随着解码器的修改， `TimeClientHandler` 不再使用 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) :

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

是不是简单优雅了很多？服务端用同样的方法修改。首先修改 `TimeServerHandler` ：

```
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```

现在唯一缺乏的部分就是编码器了，实现 [`ChannelOutboundHandler`](http://netty.io/4.0/api/io/netty/channel/ChannelOutboundHandler.html) ，将一个 `UnixTime` 对象转换为 [`ByteBuf`](http://netty.io/4.0/api/io/netty/buffer/ByteBuf.html) 。写一个编码器很简单，因为编码消息的时候，不需要处理拆包组包的问题。

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

1. 本行中有几个重要的事情。  
第一，通过 [`ChannelPromise`] , Netty 可以标记将编码过的消息写出去的失败或者成功状态。  
第二，不需要调用 `ctx.flush()` 。有一个专门的处理器方法 `void flush(ChannelHandlerContext ctx)` 用来覆盖 `flush()` 操作。

想要更简单，你可以使用 [`MessageToByteEncoder`](http://netty.io/4.0/api/io/netty/handler/codec/MessageToByteEncoder.html) :

```java
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int)msg.value());
    }
}
```

最后一步是在服务端将 `TimeEncoder` 放进 [`ChannelPipeline`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html) ,插在 `TimeServerHandler` 之前。这只是一个微不足道的小工作了。

## 关闭你的应用 [Shutting Down Your Application](http://netty.io/wiki/user-guide-for-4.x.html#shutting-down-your-application)

关闭一个 Netty 应用很简单，通过 `shutdownGracefully()` 方法将你创建的所有 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 。当所有的 [`EventLoopGroup`](http://netty.io/4.0/api/io/netty/channel/EventLoopGroup.html) 都完全关闭，并且相应群组中的 [`Channel`](http://netty.io/4.0/api/io/netty/channel/Channel.html) 都已经关闭的时候，它会返回一个 [`Future`](http://netty.io/4.0/api/io/netty/util/concurrent/Future.html) 对象通知你。

## 总结

在本章，我们通过展示如何用 Netty 编写一个完整的网络应用，快速的浏览了一次 Netty。

在接下来的章节里，有关于 Netty 的更多呃详细信息。我们也鼓励你浏览 [`io.netty.example`](https://github.com/netty/netty/tree/4.0/example/src/main/java/io/netty/example) 包中的例子。

[社区](http://netty.io/community.html) 随时等待解决你的问题，也一直期待你的想法和反馈，改善 Netty 和文档。



>如有问题，请提 [issue](https://github.com/mahui/netty-user-guide-for-4.x-CN/issues)
