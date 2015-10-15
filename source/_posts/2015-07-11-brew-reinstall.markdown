---
layout: post
title: "重新安装Homebrew"
date: 2015-07-11 23:59:30 +0800
published: true
comments: true
categories: [mac,brew]
tags: [mac,brew]
keywords: macosx, homebrew, brew
description: Macosx下重新安装Homebrew
---

![Homebrew](/images/blog/homebrew.png)

Mac Book用了几年之后，基本上其他的笔记本都已经看不上了，这个看不上不仅仅是设计感、硬件性价比等原因，更多是对工作效率的提高（尤其对于一个多年的互联网工作者），工作效率极有可能会成为一个公司成功或失败的一个不可忽视的原因。[Homebrew](http://brew.sh/)是我在Mac下一直使用的包管理系统，而且我觉得[Homebrew](http://brew.sh)是提高工作效率的非常重要的组成部分。

前段时间给[媳妇](http://janehao.github.io)也买了一台13寸的Mac Book Pro，当天就配置好了[Homebrew](http://brew.sh)环境，但是不知怎么回事，今天在使用brew时，提示下面的错误信息：


```bash
zsh: command not found: brew
```

进入brew的工作目录`/usr/local`一看，原来`/usr/local/bin`都没有了，难怪系统找不到brew这个命令。

那么怎么办？最简单的办法就是重新安装[Homebrew](http://bash.sh)。

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

但是执行安装命令之后，发现提示brew已经安装了。


查看`/usr/local`目录发现，原来还有一个关键目录还在，就是`.git`目录。用过Git的同学都应知道，`.git`目录是你本地仓库的，存储了所有该仓库的信息和历史。安装脚本检测到有brew之前的信息，当然会提示你brew已经安装了。


```
$ ls -al /usr/local/
drwxrwxr-x  15 root      admin   510B  7 11 23:50 .
drwxr-xr-x@ 12 root      wheel   408B  3  7 22:59 ..
drwxr-xr-x  14 tonydeng  admin   476B  7 12 00:17 .git
-rw-r--r--   1 tonydeng  admin   301B  7 11 23:50 .gitignore
-rw-r--r--   1 tonydeng  admin   261B  7 11 23:50 .yardopts
drwxr-xr-x   4 tonydeng  admin   136B  7 11 23:50 share
```

好吧，找到问题关键了，那么我们就解决问题吧。


执行下面的命令，brew就回来了，又可以愉快的玩耍了~~

```
rm -rf /usr/local/.git && ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)
```
