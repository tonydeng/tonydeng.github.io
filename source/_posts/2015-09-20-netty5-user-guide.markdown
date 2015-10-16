---
layout: post
title: "Netty 5用户指南"
date: 2015-09-20 21:03:35 +0800
published: true
comments: true
categories: [java, netty]
tags: [java, netty]
keywords: java netty
description:  Netty 5用户指南
---

Netty是一个`NIO client/server`网络编程框架，可以快速且容易的开发基于协议（protocol）的网络客户端/服务端的程序，它极大简化了TCP和UDP socket编程，提供流水线化的操作。

“快速且容易”并不会导致应用难以维护和性能低下。Netty经过仔细设计，基于许多协议实现的经验如：`FTP`、`SMTP`、`HTTP`以及各种二进制和基于本文的老的协议，Netty成功的找到一种保证开发，稳定性，灵活性的方式，毋需折中。

![netty components](/images/blog/netty/components.png)

# 特性

## 设计
* 为各种传输类型提供统一的API，阻塞和非阻塞的socket
* 灵活和可扩展的事件模型，将关注点分离
* 高度可定制的线程模型-单线程，一种或多个线程池，如[SEDA](http://en.wikipedia.org/wiki/Staged_event-driven_architecture)
* 真正无连接的数据报socket支持（since 3.1）

## 易用

* 提供Javadoc，用户指南和例子
* JDK1.5以上版本不需要第三方库支持

## 性能
* 更好的吞吐率，低延迟
* 较少的资源占用
* 最小化不必要的内存拷贝

## 安全
*  完整的SSL/TLS， StartTLS支持

## 社区
* 作者从2003就开始写类似的框架（`mina`），一直收集用户反馈。官方的[user guide](http://netty.io/wiki/user-guide-for-5.x.html)，中文翻译的[用户指南](http://www.getsetter.cn/2014/07/15/netty5/)

# 前言

## 问题

现如今我们使用通用的应用程序或者类库来实现系统之间的互相访问，比如，我们经常使用一个HTTP客户端来从web服务器上获取信息，或者通过web service来执行一个远程的调用。

然而，一个通用的协议和它的实现有时候并没用覆盖到一些场景。比如我们无法使用通用的http服务器来对庞大的文件和电子邮件进行交互，再比如我们无法处理那中准实时消息，比如财务信息和多人游戏数据。我们需要一个高效的协议实现用来处理一些特殊的场景。例如你可以实现一个基于AJAX的聊天应用，媒体流传输或者大文件传输的http服务器，你甚至可以想自己设计一个新的协议来准确定制你自己的需求。

另外，不可避免的事情是你不得不处理这些私有协议来确保和原有系统的互通。在这个例子中会告诉你实现一个对应该程序不损耗稳定性和性能的协议是多么的快。

## 解决方案

[Netty](http://netty.io)是一个提供异步事件驱动的网络应用框架，可以为服务端和客户端提供一个快速迭代开发的高性能、高伸缩性的协议。

换句话说，Netty是一个能够快速简单地来开发网络应用程序的NIO框架比如客户端和服务端的协议，Netty大大简化了网络程序的开发过程比如TCP和UDP的 Socket的开发。


快速和简单并不等于一个应用会产生可维护性和性能的问题，Netty是一个被精心设计的框架，他从许多协议的实现中吸收了很多的经验比如FTP,SMTP,HTTP和许多二进制和基于文本的传统协议，Netty已经成功地找出了解决方案并且在不妥协开发效率、性能、稳定性、灵活性情况下。

有一些用户可能已经发现其他的一些网络框架声称自己也有同样的优势，所以你可能会问是Netty和他们有什么不一样。答案就是Netty的哲学设计理念。Netty从第一天开始就把API和实现设计成为你提供最舒适的使用体验。Netty的设计理念会使当你阅读本指南并且和Netty打交道的时候，让你的使用变得更加简单。

# 入门指南

这个章节会围绕Netty核心的结构并附上一些简单的例子让你快速的入门。当你读完本章节你马上就可以用Netty写出一个客户端和一个服务端。
如果你在学习某些东西的时候喜欢自顶向下的方法，你可能想要从第二个章节架构概述开始，然后再回到这里。

## 开始之前

如果要运行本章节中的2个例子最低要求是：Netty的最新版本(Netty5)和JDK1.6及以上。最新的Netty版本在项目[下载页面](http://netty.io/downloads.html)。为了下载到正确的JDK版本，请到你偏爱的网站下载。
当你在读的过程中，你可能有很多关于本章节介绍的类相关的问题，每当你想知道他们的更多的信息那么请参考API说明文档。为了方便，所有文档中涉及到的类名字都会被关联到一个在线的API说明。当然如果有任何错误信息、语法错误或者你有任何好的建议来改进文档说明，那么请联系[netty社区](http://netty.io/community.html)。

## 实现Discard服务器

世界上最简单的协议不是”Hello,World!”，是[DISCARD](http://tools.ietf.org/html/rfc863)，他是一种丢弃所有接收的数据，并不做有任何响应的协议。
为了实现DISCARD协议，你唯一需要做的就是忽略所有收到的数据。让我们从处理器的实现开始，处理器是由Netty生成用来处理I/O事件的。

```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

* DisCardServerHandler 继承自 [ChannelHandlerAdapter](http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAdapter.html)，这个类实现了[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)接口，ChannelHandler提供了许多事件处理的接口方法，然后你可以覆盖这些方法。现在仅仅只需要继承[ChannelHandlerAdapter](http://netty.io/5.0/api/io/netty/channel/ChannelHandlerAdapter.html)类而不是你自己去实现接口方法。

* 这里我们覆盖了`chanelRead()`事件处理方法。每当从客户端收到新的数据时，这个方法会在收到消息时被调用，这个例子中，收到的消息的类型是[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)

* 为了实现DISCARD协议，处理器不得不忽略所有接受到的消息。[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)是一个引用计数对象，这个对象必须显示地调用release()方法来释放。请记住处理器的职责是释放所有传递到处理器的引用计数对象。通常，channelRead()方法的实现就像下面的这段代码：

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```
*  `exceptionCaught()`事件处理方法是当出现Throwable对象才会被调用，即当Netty由于IO错误或者处理器在处理事件时抛出的异常时。在大部分情况下，捕获的异常应该被记录下来并且把关联的channel给关闭掉。然而这个方法的处理方式会在遇到不同异常的情况下有不同的实现，比如你可能想在关闭连接之前发送一个错误码的响应消息。


到目前为止一切都还比较顺利，我们已经实现了DISCARD服务的一半功能，剩下的需要编写一个`main()`方法来启动服务端的DiscardServerHandler。

```
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
 * Discards any incoming data.
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

            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)

            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
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


* [NioEventLoopGroup](http://netty.io/5.0/api/io/netty/channel/nio/NioEventLoopGroup.html) 是用来处理I/O操作的多线程事件循环器，Netty提供了许多不同的[EventLoopGroup](http://netty.io/5.0/api/io/netty/channel/EventLoopGroup.html)的实现用来处理不同传输协议。在这个例子中我们实现了一个服务端的应用，因此会有2个[NioEventLoopGroup](http://netty.io/5.0/api/io/netty/channel/nio/NioEventLoopGroup.html)会被使用。第一个经常被叫做‘`boss`’，用来接收进来的连接。第二个经常被叫做‘`worker`’，用来处理已经被接收的连接，一旦‘`boss`’接收到连接，就会把连接信息注册到‘`worker`’上。如何知道多少个线程已经被使用，如何映射到已经创建的[Channels](http://netty.io/5.0/api/io/netty/channel/Channel.html)上都需要依赖于[EventLoopGroup](http://netty.io/5.0/api/io/netty/channel/nio/NioEventLoopGroup.html)的实现，并且可以通过构造函数来配置他们的关系。
* [ServerBootstrap](http://netty.io/5.0/api/io/netty/bootstrap/ServerBootstrap.html) 是一个启动NIO服务的辅助启动类。你可以在这个服务中直接使用[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)，但是这会是一个复杂的处理过程，在很多情况下你并不需要这样做。
* 这里我们指定使用[NioServerSocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)类来举例说明一个新的[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)如何接收进来的连接。
* 这里的事件处理类经常会被用来处理一个最近的已经接收的[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)。[ChannelInitializer](http://netty.io/5.0/api/io/netty/channel/ChannelInitializer.html)是一个特殊的处理类，他的目的是帮助使用者配置一个新的[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)。也许你想通过增加一些处理类比如DiscardServerHandle来配置一个新的[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)或者其对应的[ChannelPipeline](http://netty.io/5.0/api/io/netty/channel/ChannelPipeline.html)来实现你的网络程序。当你的程序变的复杂时，可能你会增加更多的处理类到`pipline`上，然后提取这些匿名类到最顶层的类上。
*  你可以设置这里指定的通道实现的配置参数。我们正在写一个TCP/IP的服务端，因此我们被允许设置socket的参数选项比如`tcpNoDelay`和`keepAlive`。请参考[ChannelOption](http://netty.io/5.0/api/io/netty/channel/ChannelOption.html)和详细的[ChannelConfig](http://netty.io/5.0/api/io/netty/channel/ChannelConfig.html)实现的接口文档以此可以对[ChannelOptions](http://netty.io/5.0/api/io/netty/channel/ChannelOption.html)的有一个大概的认识。
* 你关注过`option()`和`childOption()`吗？`option()`是提供给[NioServerSocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)用来接收进来的连接。childOption()是提供给由父管道[ServerChannel](http://netty.io/5.0/api/io/netty/channel/ServerChannel.html)接收到的连接，在这个例子中也是[NioServerSocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)。
* 我们继续，剩下的就是绑定端口然后启动服务。这里我们在机器上绑定了机器所有网卡上的8080端口。当然现在你可以多次调用`bind()`方法(基于不同绑定地址)。

恭喜！你已经完成熟练地完成了第一个基于NETTY的服务端程序。

### 观察接收到的数据

现在我们已经编写出我们第一个服务端，我们需要测试一下他是否真的可以运行。最简单的测试方法是用`telnet` 命令。例如，你可以在命令行上输入`telnet localhost 8080`或者其他类型参数。

然而我们能说这个服务端是正常运行了吗？事实上我们也不知道因为他是一个discard服务，你根本不可能得到任何的响应。为了证明他仍然是在工作的，让我们修改服务端的程序来打印出他到底接收到了什么。

我们已经知道`channelRead()`方法是在数据被接收的时候调用。让我们放一些代码到DiscardServerHandler类的`channelRead()`方法。

```
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

* 这个低效的循环事实上可以简化为:`System.out.println(in.toString(io.netty.util.CharsetUtil.US_ASCII))`

* 或者，你可以在这里调用`in.release()`。

如果你再次运行`telnet`命令，你将会看到服务端打印出了他所接收到的消息。

完整的discard server代码放在了[io.netty.example.discard](http://netty.io/5.0/xref/io/netty/example/discard/package-summary.html)包下面。

## ECHO服务（响应式协议）

到目前为止，我们虽然接收到了数据，但没有做任何的响应。然而一个服务端通常会对一个请求作出响应。让我们学习怎样在[ECHO](http://tools.ietf.org/html/rfc862)协议的实现下编写一个响应消息给客户端，这个协议针对任何接收的数据都会返回一个响应。

和discard server唯一不同的是把在此之前我们实现的`channelRead()`方法，返回所有的数据替代打印接收数据到控制台上的逻辑。因此，需要把`channelRead()`方法修改如下：

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ctx.write(msg); // (1)
    ctx.flush(); // (2)
}
```


* [ChannelHandlerContext](http://netty.io/5.0/api/io/netty/channel/ChannelHandlerContext.html,)对象提供了许多操作，使你能够触发各种各样的I/O事件和操作。这里我们调用了`write(Object)`方法来逐字地把接受到的消息写入。请注意不同于DISCARD的例子我们并没有释放接受到的消息，这是因为当写入的时候NETTY已经帮我们释放了。
* `ctx.write(Object)`方法不会使消息写入到通道上，他被缓冲在了内部，你需要调用`ctx.flush()`方法来把缓冲区中数据强行输出。或者你可以用更简洁的`cxt.writeAndFlush(msg)`以达到同样的目的。

如果你再一次运行telnet命令，你会看到服务端会发回一个你已经发送的消息。
完整的echo服务的代码放在了[io.netty.example.echo](http://netty.io/5.0/xref/io/netty/example/echo/package-summary.html)包下面。

## TIME服务(时间协议的服务)

在这个部分被实现的协议是[TIME](http://tools.ietf.org/html/rfc868)协议。和之前的例子不同的是在不接受任何请求时他会发送一个含32位的整数的消息，并且一旦消息发送就会立即关闭连接。在这个例子中，你会学习到如何构建和发送一个消息，然后在完成时主动关闭连接。

因为我们将会忽略任何接收到的数据，而只是在连接被创建发送一个消息，所以这次我们不能使用`channelRead()`方法了，代替他的是，我们需要覆盖`channelActive()`方法，下面的就是实现的内容：

```
package io.netty.example.time;

public class TimeServerHandler extends ChannelHandlerAdapter {

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

* `channelActive()`方法将会在连接被建立并且准备进行通信时被调用。因此让我们在这个方法里完成一个代表当前时间的32位整数消息的构建工作。
* 为了发送一个新的消息，我们需要分配一个包含这个消息的新的缓冲。因为我们需要写入一个32位的整数，因此我们需要一个至少有4个字节的[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)。通过`ChannelHandlerContext.alloc()`得到一个当前的ByteBufAllocator，然后分配一个新的缓冲。
* 和往常一样我们需要编写一个构建好的消息。但是等一等，flip在哪？难道我们使用NIO发送消息时不是调用`java.nio.ByteBuffer.flip()`吗？ByteBuf之所以没有这个方法因为有两个指针，一个对应读操作一个对应写操作。当你向ByteBuf里写入数据的时候写指针的索引就会增加，同时读指针的索引没有变化。读指针索引和写指针索引分别代表了消息的开始和结束。

比较起来，NIO缓冲并没有提供一种简洁的方式来计算出消息内容的开始和结尾，除非你调用`flip`方法。当你忘记调用flip方法而引起没有数据或者错误数据被发送时，你会陷入困境。这样的一个错误不会发生在Netty上，因为我们对于不同的操作类型有不同的指针。你会发现这样的使用方法会让你过程变得更加的容易，因为你已经习惯一种没有使用`flip`的方式。

另外一个点需要注意的是`ChannelHandlerContext.write()`(和`writeAndFlush()`)方法会返回一个[ChannelFuture](http://netty.io/5.0/api/io/netty/channel/ChannelFuture.html)对象，一个[ChannelFuture](http://netty.io/5.0/api/io/netty/channel/ChannelFuture.html)代表了一个还没有发生的I/O操作。这意味着任何一个请求操作都不会马上被执行，因为在Netty里所有的操作都是异步的。举个例子下面的代码中在消息被发送之前可能会先关闭连接。


```
Channel ch = ...;
ch.writeAndFlush(message);
ch.close();
```

因此你需要在`write()`方法返回的ChannelFuture完成后调用`close()`方法，然后当他的写操作已经完成他会通知他的监听者。请注意,`close()`方法也可能不会立马关闭，他也会返回一个[ChannelFuture](http://netty.io/5.0/api/io/netty/channel/ChannelFuture.html)。

* 当一个写请求已经完成是如何通知到我们？这个只需要简单地在返回的ChannelFuture上增加一个[ChannelFutureListener](http://netty.io/5.0/api/io/netty/channel/ChannelFutureListener.html)。这里我们构建了一个匿名的[ChannelFutureListener](http://netty.io/5.0/api/io/netty/channel/ChannelFutureListener.html)类用来在操作完成时关闭Channel。

或者，你可以使用简单的预定义监听器代码:

```
f.addListener(ChannelFutureListener.CLOSE);
```
为了测试我们的time服务如我们期望的一样工作，你可以使用UNIX的`rdate`命令

```
$ rdate -o <port> -p <host>
```

port是你在`main()`函数中指定的端口，host使用locahost就可以了。

## Time客户端

不像DISCARD和ECHO的服务端，对于TIME协议我们需要一个客户端因为人们不能把一个32位的二进制数据翻译成一个日期或者日历。在这一部分，我们将会讨论如何确保服务端是正常工作的，并且学习怎样用NETTY编写一个客户端。

在Netty中,编写服务端和客户端最大的并且唯一不同的使用了不同的[BootStrap](http://netty.io/5.0/api/io/netty/bootstrap/Bootstrap.html)和[Channel](http://netty.io/5.0/api/io/netty/channel/Channel.html)的实现。请看一下下面的代码：

```
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

            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

* [BootStrap](http://netty.io/5.0/api/io/netty/bootstrap/Bootstrap.html)和[ServerBootstrap](http://netty.io/5.0/api/io/netty/bootstrap/ServerBootstrap.html)类似,不过他是对非服务端的channel而言，比如客户端或者无连接传输模式的channel。
* 如果你只指定了一个[EventLoopGroup](http://netty.io/5.0/api/io/netty/channel/EventLoopGroup.html)，那他就会即作为一个‘`boss`’线程，也会作为一个‘`workder`’线程，尽管客户端不需要使用到‘`boss`’线程。
* 代替[NioServerSocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/nio/NioServerSocketChannel.html)的是[NioSocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/nio/NioSocketChannel.html),这个类在客户端channel被创建时使用。
* 不像在使用ServerBootstrap时需要用`childOption()`方法，因为客户端的[SocketChannel](http://netty.io/5.0/api/io/netty/channel/socket/SocketChannel.html)没有父channel的概念。
* 我们用`connect()`方法代替了`bind()`方法。
正如你看到的，他和服务端的代码是不一样的。[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)是如何实现的?他应该从服务端接受一个32位的整数消息，把他翻译成人们能读懂的格式，并打印翻译好的时间，最后关闭连接:

```
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelHandlerAdapter {
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


* 在TCP/IP中，NETTY会把读到的数据放到ByteBuf的数据结构中。

这样看起来非常简单，并且和服务端的那个例子的代码也相差不多。然而，处理器有时候会因为抛出`IndexOutOfBoundsException`而拒绝工作。在下个部分我们会讨论为什么会发生这种情况。


# 流数据的传输处理


## 一个小的Socket Buffer问题

在基于流的传输里比如TCP/IP，接收到的数据会先被存储到一个socket接收缓冲里。不幸的是，基于流的传输并不是一个数据包队列，而是一个字节队列。即使你发送了2个独立的数据包，操作系统也不会作为2个消息处理而仅仅是作为一连串的字节而言。因此这是不能保证你远程写入的数据就会准确地读取。举个例子，让我们假设操作系统的TCP/TP协议栈已经接收了3个数据包：

![netty socket buffer](/images/blog/netty/netty5_1.png)

由于基于流传输的协议的这种普通的性质，在你的应用程序里读取数据的时候会有很高的可能性被分成下面的片段。

![netty socket buffer](/images/blog/netty/netty5_2.png)

因此，一个接收方不管他是客户端还是服务端，都应该把接收到的数据整理成一个或者多个更有意思并且能够让程序的业务逻辑更好理解的数据。在上面的例子中，接收到的数据应该被构造成下面的格式：

![netty socket buffer](/images/blog/netty/netty5_3.png)

### 第一个解决方案

现在让我们回到TIME客户端的例子上。这里我们遇到了同样的问题，一个32字节数据是非常小的数据量，他并不见得会被经常拆分到到不同的数据段内。然而，问题是他确实可能会被拆分到不同的数据段内，并且拆分的可能性会随着通信量的增加而增加。

最简单的方案是构造一个内部的可积累的缓冲，直到4个字节全部接收到了内部缓冲。下面的代码修改了TimeClientHandler的实现类修复了这个问题

```
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelHandlerAdapter {
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
            long currentTimeMillis = (buf.readInt() - 2208988800L) * 1000L;
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


* [ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)有2个生命周期的监听方法：handlerAdded()和handlerRemoved()。你可以完成任意初始化任务只要他不会被阻塞很长的时间。
* 首先，所有接收的数据都应该被累积在buf变量里。
* 然后，处理器必须检查buf变量是否有足够的数据，在这个例子中是4个字节，然后处理实际的业务逻辑。否则，Netty会重复调用channelRead()当有更多数据到达直到4个字节的数据被积累。


### 第二个解决方案

尽管第一个解决方案已经解决了Time客户端的问题了，但是修改后的处理器看起来不那么的简洁，想象一下如果由多个字段比如可变长度的字段组成的更为复杂的协议时，你的[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)的实现将很快地变得难以维护。

正如你所知的，你可以增加多个[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)到[ChannelPipeline](http://netty.io/5.0/api/io/netty/channel/ChannelPipeline.html) ,因此你可以把一整个ChannelHandler拆分成多个模块以减少应用的复杂程度，比如你可以把TimeClientHandler拆分成2个处理器：

* TimeDecoder处理数据拆分的问题
* TimeClientHandler原始版本的实现

幸运地是，Netty提供了一个可扩展的类，帮你完成TimeDecoder的开发。

```
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

* [ByteToMessageDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)是[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)的一个实现类，他可以在处理数据拆分的问题上变得很简单。
* 每当有新数据接收的时候，[ByteToMessageDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)都会调用decode()方法来处理内部的那个累积缓冲。
* `Decode()`方法可以决定当累积缓冲里没有足够数据时可以往out对象里放任意数据。当有更多的数据被接收了[ByteToMessageDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)会再一次调用`decode()`方法。
* 如果在`decode()`方法里增加了一个对象到out对象里，这意味着解码器解码消息成功。[ByteToMessageDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)将会丢弃在累积缓冲里已经被读过的数据。请记得你不需要对多条消息调用`decode()`，[ByteToMessageDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ByteToMessageDecoder.html)会持续调用`decode()`直到不放任何数据到out里。

现在我们有另外一个处理器插入到[ChannelPipeline](http://netty.io/5.0/api/io/netty/channel/ChannelPipeline.html)里，我们应该在TimeClient里修改[ChannelInitializer](http://netty.io/5.0/api/io/netty/channel/ChannelInitializer.html) 的实现：

```
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果你是一个大胆的人，你可能会尝试使用更简单的解码类[ReplayingDecoder](http://netty.io/5.0/api/io/netty/handler/codec/ReplayingDecoder.html)。不过你还是需要参考一下API文档来获取更多的信息。

```
public class TimeDecoder extends ReplayingDecoder {
@Override
    protected void decode(
        ChannelHandlerContext ctx, ByteBuf in, List out) {
        out.add(in.readBytes(4));
    }
}
```

此外，Netty还提供了更多可以直接拿来用的解码器使你可以更简单地实现更多的协议，帮助你避免开发一个难以维护的处理器实现。请参考下面的包以获取更多更详细的例子：

* 对于二进制协议请看[io.netty.example.factorial](http://netty.io/5.0/xref/io/netty/example/factorial/package-summary.html)
* 对于基于文本协议请看[io.netty.example.telnet](http://netty.io/5.0/xref/io/netty/example/telnet/package-summary.html)


## 用POJO代替ByteBuf

我们已经讨论了所有的例子，到目前为止一个消息的消息都是使用[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)作为一个基本的数据结构。在这一部分，我们会改进TIME协议的客户端和服务端的例子，用POJO替代[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)。

在你的[ChannelHandlerS](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)中使用POJO优势是比较明显的。通过从[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)中提取出ByteBuf的代码，将会使[ChannelHandler](http://netty.io/5.0/api/io/netty/channel/ChannelHandler.html)的实现变得更加可维护和可重用。在TIME客户端和服务端的例子中，我们读取的仅仅是一个32位的整形数据，直接使用ByteBuf不会是一个主要的问题。然后，你会发现当你需要实现一个真实的协议，分离代码变得非常的必要。

首先，让我们定义一个新的类型叫做UnixTime。

```
package io.netty.example.time;

import java.util.Date;

public class UnixTime {

    private final int value;

    public UnixTime() {
        this((int) (System.currentTimeMillis() / 1000L + 2208988800L));
    }

    public UnixTime(int value) {
        this.value = value;
    }

    public int value() {
        return value;
    }

    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}
```

现在我们可以修改下TimeDecoder类，返回一个UnixTime，以替代[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)

```
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readInt()));
}
````

下面是修改后的解码器，TimeClientHandler不再有任何的[ByteBuf](http://netty.io/5.0/api/io/netty/buffer/ByteBuf.html)代码了。

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```

是不是变得更加简单和优雅了？相同的技术可以被运用到服务端。让我们修改一下TimeServerHandler的代码。

```
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```

现在，仅仅需要修改的是ChannelHandler的实现，这里需要把UnixTime对象重新转化为一个ByteBuf。不过这已经是非常简单了，因为当你对一个消息编码的时候，你不需要再处理拆包和组装的过程。

```
package io.netty.example.time;

public class TimeEncoder extends ChannelHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt(m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

在这几行代码里还有几个重要的事情。

* 第一， 通过ChannelPromise，当编码后的数据被写到了通道上Netty可以通过这个对象标记是成功还是失败。
* 第二， 我们不需要调用`cxt.flush()`。因为处理器已经单独分离出了一个方法`void flush(ChannelHandlerContext cxt)`,如果像自己实现`flush`方法内容可以自行覆盖这个方法。


进一步简化操作，你可以使用[MessageToByteEncode](http://netty.io/5.0/api/io/netty/handler/codec/MessageToByteEncoder.html):

```
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt(msg.value());
    }
}
```

最后的任务就是在TimeServerHandler之前把TimeEncoder插入到[ChannelPipeline](http://netty.io/5.0/api/io/netty/channel/ChannelPipeline.html)。但这是不那么重要的工作。

## 关闭你的应用

关闭一个Netty应用往往只需要简单地通过`shutdownGracefully()`方法来关闭你构建的所有的[NioEventLoopGroup](http://netty.io/5.0/api/io/netty/channel/EventLoopGroup.html)S.当[EventLoopGroup](http://netty.io/5.0/api/io/netty/channel/EventLoopGroup.html)被完全地终止,并且对应的所有channels都已经被关闭时，Netty会返回一个[Future](http://netty.io/5.0/api/io/netty/util/concurrent/Future.html)对象。

## 总结

在这一章节中，我们会快速地回顾下如果在熟练掌握Netty的情况下编写出一个健壮能运行的网络应用程序。
在Netty接下去的章节中还会有更多更相信的信息。我们也鼓励你去重新复习下在[io.netty.example](https://github.com/netty/netty/tree/master/example/src/main/java/io/netty/example)包下的例子。
请注意[社区](http://netty.io/community.html)一直在等待你的问题和想法以帮助Netty的持续改进，Netty的文档也是基于你们的快速反馈上。
