---
layout: post
title: "Linux下文件系统结构与权限"
date: 2014-12-26 17:36:54 +0800
published: false
comments: true
categories: [linux]
tags: [linux]
keywords: linux, filesystem, permissions, 文件系统, 权限
description: Linux下文件系统结构与权限
---

## 人生若只如初见

我们在初次接触Linux时，最常见的困扰经常会使这样的：

* 咦？我的C盘在哪儿啊？
* 我怎么知道哪些程序是可以执行？
* 为什么这些可执行的程序不是.exe的呢？

> 针对这些困扰的问题，我们来看看Linux的文件系统结构及权限。

## Linux树状文件系统结构

![Linux树状文件系统结构](/images/blog/linux_filesystem_tree.jpg)
