layout: post
title: 使用 lsof 代替 Mac OS X 中的 netstat 查看占用端口的程序
published: true
comments: true
sharing: true
footer: true
categories: [Mac OSX]
tags: [mac, lsof, netstat]
date: 2016-07-07 13:06:48
keywords: mac lsof netstat
---


众所周知水果系统内核是有 BSD 血统的 `Darwin`，OS X 自带的很多 CLI 工具也是 BSD style 的，有一部分使用起来和 Linux 无异，有一部分可以通过 `brew` 安装 GNU 版本（如 `tar`），但是 OS X 的 `netstat` 不能查看使用端口的程序名让我一直很不爽，而且也没找到 GNU 版本，于是去搜了一下解决办法，stackoverflow 上的结论基本都是建议使用 `lsof` 代替 `netstat` 进行查看：

```
sudo lsof -nP -iTCP:端口号 -sTCP:LISTEN
```
<!-- more -->
* -n 表示不显示主机名
* -P 表示不显示端口俗称
* 不加 sudo 只能查看以当前用户运行的程序

另外，还可以通过管道来过滤想要的信息

```
sudo lsof -nP -iTCP -sTCP:LISTEN | grep python
```

基本效果如下：

查看当前所有监听的端口以及对应的`Command`和`PID`

```
➜  ~ lsof -nP -iTCP -sTCP:LISTEN
COMMAND    PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
SSH\x20Pr 1553 tonydeng    8u  IPv4 0xee7327e39355d175      0t0  TCP 127.0.0.1:8087 (LISTEN)
SSH\x20Pr 1553 tonydeng    9u  IPv6 0xee7327e38aad6e15      0t0  TCP [::1]:8087 (LISTEN)
java      2978 tonydeng  166u  IPv6 0xee7327e38aad7e35      0t0  TCP *:62622 (LISTEN)
node      3319 tonydeng   31u  IPv4 0xee7327e39f0f8745      0t0  TCP *:4000 (LISTEN)
```

查看指定端口对应的`Command`和`PID`

```
➜  ~ lsof -nP -iTCP:4000 -sTCP:LISTEN
COMMAND  PID     USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
node    3319 tonydeng   31u  IPv4 0xee7327e39f0f8745      0t0  TCP *:4000 (LISTEN)
```

PS ： 输出占用该端口的 PID

```
lsof -nP -iTCP:4000 |grep LISTEN|awk '{print $2;}'
```

