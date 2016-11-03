layout: post
title: etcd4j阻塞排查和解除过程
published: true
comments: true
sharing: true
footer: true
categories: [程序设计]
tags: [etcd, etcd4j, java]
date: 2016-11-03 13:19:42
keywords:
---

![debug](/images/blog/etcd4j/debug.jpg)

## 先说一下背景

最近将整个微服务的体系从原来的通过使用[TCP4J](https://github.com/tonydeng/tcp4j)加上固定的`HOST+PORT`的方式转成使用[ETCD](https://github.com/coreos/etcd)的服务发现体系（大家想了解ETCD可以关注我之前的两篇博文[初试ETCD](/2015/11/24/etcd-the-first-using/),[ETCD应用场景 ](/2015/10/19/etcd-application-scenarios/)）。


### 基于ETCD的注册 & 服务发现架构

![etcd registry](/images/blog/etcd4j/etcd-registry.png)

1. etcd registry 做为服务中心，提供注册与服务发现。
2. 资源服务在准备完毕之后将服务实例注册到服务中心。
3. 客户端到服务注册中心根据服务名称获取资源服务的地址。
4. 客户端获取资源服务的地址后，调用资源服务。
5. 资源服务在关闭时需要将服务实例在服务中心进行注销操作。

我们使用[etcd4j](https://github.com/jurmous/etcd4j)这个etcd的java client实现，在使用的过程中，etcd4j-client会首先去请求一下etcd server，看看当前etcd server的版本是多少。

比如这样：

```
http http://etcd.dev.cim.in:2379/version
HTTP/1.1 200 OK
Content-Length: 44
Content-Type: application/json
Date: Wed, 02 Nov 2016 08:21:34 GMT

{
   "etcdcluster": "2.3.0",
   "etcdserver": "2.3.7"
}
```

## 看看问题

看起来貌似很简单的情况，但是在改造某个项目的时候，在这个简单的场景下，出现了一个问题，让@webwyz同学头疼了一下午。

在启动项目时，就停在get version这儿死活过不去了。

日志如下：

```java
2016-11-02 17:45:06 DEBUG m.etcd4j.transport.EtcdNettyClient - Connected to etcd.dev.cim.in/192.168.1.86:2379 (0)
2016-11-02 17:45:06 DEBUG i.n.u.i.JavassistTypeParameterMatcherGenerator - Generated: io.netty.util.internal.__matchers__.io.netty.handler.codec.http.FullHttpResponseMatcher
2016-11-02 17:45:07 DEBUG m.e.transport.EtcdResponseHandler - Received 200 for GET /version
```

正确的启动日志应该如此：

```java
2016-11-02 15:59:05 DEBUG m.etcd4j.transport.EtcdNettyClient - Connected to etcd.dev.cim.in/192.168.1.86:2379 (0)
2016-11-02 15:59:05 DEBUG m.e.transport.EtcdResponseHandler - Received 200 for GET /version
2016-11-02 15:59:05 DEBUG m.etcd4j.transport.EtcdNettyClient - Connection closed for request GET on uri /version
2016-11-02 15:59:05 INFO  c.c.a.k.etcd.EtcdAutoConfiguration - etcd version is 2.3.0 , urls are [ http://etcd.dev.cim.in:2379 ]
.......
```

## 排查异常

### 是否是网络连接问题？

当时我查看网络状况，网络连接也是正常的。

```
lsof -nP -iTCP:2379
COMMAND   PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java     7142 tonydeng  317u  IPv6 0x235c7b856fde6855      0t0  TCP 192.168.2.231:50774->192.168.1.86:2379 (ESTABLISHED)
java    12179 tonydeng  293u  IPv6 0x235c7b857295c315      0t0  TCP 192.168.2.231:52504->192.168.1.86:2379 (ESTABLISHED)
```

甚至通过tcpdump来进行抓包也没有发现什么问题。

```
sudo tcpdump host 192.168.2.231 and 192.168.1.86 and tcp port 2379 -vv
```

看来不是在网络层面的问题。

### 追追代码

从日志上看，最后的输出应该是[mousio.etcd4j.transport.EtcdResponseHandler](https://github.com/jurmous/etcd4j/blob/release-2.12.0/src/main/java/mousio/etcd4j/transport/EtcdResponseHandler.java#L100)，先看看这个类的代码(有兴趣的同学，可以在Github上查看[相关代码](https://github.com/jurmous/etcd4j/blob/release-2.12.0/src/main/java/mousio/etcd4j/transport/EtcdResponseHandler.java)，就不列出所有代码了)。

```java
        if(logger.isDebugEnabled()) {
            logger.debug("Received {} for {} {}", new Object[]{Integer.valueOf(status.code()), this.request.getMethod().name(), this.request.getUri()});
        }

        if(!status.equals(HttpResponseStatus.MOVED_PERMANENTLY) && !status.equals(HttpResponseStatus.TEMPORARY_REDIRECT)) {
            EtcdResponseDecoder failureDecoder = (EtcdResponseDecoder)failureDecoders.get(status);
            if(failureDecoder != null) {
                this.promise.setFailure((Throwable)failureDecoder.decode(headers, content));
            } else if(!content.isReadable()) {
                if(!status.equals(HttpResponseStatus.OK) && !status.equals(HttpResponseStatus.ACCEPTED) && !status.equals(HttpResponseStatus.CREATED)) {
                    this.promise.setFailure(new IOException("Content was not readable. HTTP Status: " + status));
                }
            } else {
                try {
                    this.promise.setSuccess(this.request.getResponseDecoder().decode(headers, content));
                } catch (Exception var10) {
                    if(var10 instanceof EtcdException) {
                        this.promise.setFailure(var10);
                    } else {
                        try {
                            this.promise.setFailure((Throwable)EtcdException.DECODER.decode(headers, content));
                        } catch (Exception var9) {
                            this.promise.setFailure(var10);
                        }
                    }
                }
            }
        }
```

从代码上看，应该是阻塞在`decode`过程中了（如果大家看到上面的代码和github上不一样，请不要奇怪，这个是用过java工具反编译出来的）。

## 定位问题

```java
if(failureDecoder != null) {
    this.promise.setFailure((Throwable)failureDecoder.decode(headers, content));
}
```

如果是这样，那应该是在进行version信息的序列化时出现的问题，看看etcd4j使用的json序列化包是什么？

通过`mvn dependency:tree`命令查看，这里果然有问题。

正常启动项目的jackson相关的依赖及版本

```
[INFO] |  |  |  \- org.mousio:etcd4j:jar:2.12.0:compile
[INFO] |  |  |     +- com.fasterxml.jackson.core:jackson-core:jar:2.8.0:compile
[INFO] |  |  |     +- com.fasterxml.jackson.core:jackson-databind:jar:2.8.0:compile
[INFO] |  |  |     +- com.fasterxml.jackson.core:jackson-annotations:jar:2.8.0:compile
[INFO] |  |  |     \- com.fasterxml.jackson.module:jackson-module-afterburner:jar:2.8.0:compile
```

异常启动项目的jackson相关的依赖及版本

```
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.7.4:compile
[INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.7.0:compile
[INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.7.4:compile

[INFO] |  |  |  \- org.mousio:etcd4j:jar:2.12.0:compile
[INFO] |  |  |     \- com.fasterxml.jackson.module:jackson-module-afterburner:jar:2.8.0:compile
```

## 解决问题

既然知道，这儿有问题，那就简单了，在启动异常的项目中去掉不一致的jackson依赖，一切问题就都解决了。

> etcd4j项目中也有人提到类似的问题[i update version to 2.12.0 my programmer has been blocked #116](https://github.com/jurmous/etcd4j/issues/116)。
