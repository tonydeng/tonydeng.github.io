---
layout: post
title: "OSX下编译Tengine+SSL错误的解决办法"
date: 2015-03-09 15:59:08 +0800
published: true
comments: true
categories: [osx, nginx]
tags: [osx, nginx]
keywords: tengine osx make error
description: Tengine+ssl编译解决方案
---
![Nginx+SSL](/images/blog/nginx-ssl.jpg)

[Tengine](http://tengine.taobao.org/)（Tengine是由淘宝基于[Nignx](http://nginx.org/)开发的Web服务器）是一个非常要用的Web服务器，我基本上在测试及生产环境中都使用它来代替Nginx。不过之前都是在Linux下来编译和安装，这几天由于要调试之前已经离职的工程师的PHP项目，需要在自己的Mac配置一个可以使用的PHP开发调试环境。于是开始了OSX下的PHP+Nginx之旅。

由于我不太希望用**brew**来安装PHP和Tengine，还是使用最原始的编译安装的方式。安装PHP时基本上没有遇到任何问题，在安装Tengine时却碰到这样的错误。

原来的编译的参数

```
./configure --prefix=/usr/local/tengine-2.1.0 --with-http_ssl_module
```

configure阶段没有任何问题，但是在进行make编译的时候，却报了下面的错误。

```
src/core/ngx_crypt.c:82:5: error: 'MD5_Init' is deprecated: first deprecated in OS X 10.7 [-Werror,-Wdeprecated-declarations]
    ngx_md5_init(&md5);
    ^
```

MD5的其他函数，比如**MD5_Update**,**MD5_Final**也在其中。

万能的google大神告知了错误的原因及解决方案，由于之前在编译参数中加入对ssl的支持，需要使用MD5算法，但是由于Nginx和OpenSSL的兼容性问题，某些api已经被建议废弃了，编译时不停的报警告并最终错误，无法编译完成。

其实解决办法也挺简单的，在编译时加上```--with-cc-opt="-Wno-deprecated-declarations"```参数即可。
```
./configure --prefix=/usr/local/tengine-2.1.0 --with-http_ssl_module --with-cc-opt="-Wno-deprecated-declarations"
```

错误的原因及解决办法参见：

http://trac.nginx.org/nginx/ticket/587

https://www.ruby-forum.com/topic/2193556#1012378
