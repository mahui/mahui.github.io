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

> based on [User guide for 4.x]("https://github.com/netty/netty/wiki/User-guide-for-4.x")



# 序言 [Preface](http://netty.io/wiki/user-guide-for-4.x.html#preface)  

## 问题 [The Problem](http://netty.io/wiki/user-guide-for-4.x.html#the-problem)

如今我们使用通用的应用程序或者库来彼此通讯。比如，我们经常使用 HTTP 客户端库从远程服务器接收信息或者执行远程服务的调用。但无论如何，一个通用的协议或者它们的实现有时候可用性不是太好。这就是为什么有时候我们并不使用通用的 HTTP 协议来交换大数据、电子邮件信息以及金融数据和多人网络游戏数据这种近实时的数据。所需要的是为了一个专门的目的去实现一个高度优化的协议。比如，你可能需为了基于 AJAX 的聊天应用，媒体传输，或者大文件传输去实现一个优化的 HTTP 服务。你甚至可以为了你的需求去精确的定义并实现一个全新的协议。另外一个经常发生的情况是你必须去处理系统中遗留的专门协议去确保与旧系统的交互。这种情况下重要的是，我们如何能够快速的实现该协议，并且不会牺牲应用程序的稳定性和性能。

## 解决方案 [The Solution](http://netty.io/wiki/user-guide-for-4.x.html#the-solution)

[Netty](http://netty.io) 是一个提供了异步事件驱动 (asynchronous event-driven) 网络应用程序框架和工具的杰作，可以用来快速开发可维护的，高性能 · 高可用的服务端和客户端协议。

从另一方面来讲，Netty 是一个可以快速开发网络程序如客户端和服务端协议的 NIO 客户端/服务端框架。它大大的简化了如 TCP 和 UDP 这类套接字服务的网络应用程序的开发并优化了其开发流程。

“快速简单”并不意味着开发出的应用程序的可维护性变差或者遇到性能问题。 Netty 从大量的如 FTP、SMPT、HTTP 以及各种二进制文本传输协议的实现中获得经验，并精心设计。因此， Netty 成功的获得了可以快速开发，并且不妥协性能、稳定性以及灵活性的方法。

一些用户用户或者发现过其他一些网络应用框架也声称有以上优势，可能会想问 Netty 和它们有何不同。答案是，设计哲学。 Netty 从一开始就设计得为你提供舒适的 API 及实现。这不是什么可感知的事务，但你会意识到这些设计哲学让你的生活变得更简单，就像你开始阅读这份指南，并且开始轻松使用 Netty 。

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

目前来说一切都很好，我们实现了 `DISCARD` 服务的一般。现在剩下我们去在 `DiscardServerHandler` 中写一个 `main()` 方法来启动服务。


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

*<未完待续>*
