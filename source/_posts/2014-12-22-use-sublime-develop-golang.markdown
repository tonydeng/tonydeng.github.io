---
layout: post
title: "使用Sublime开发Golang"
date: 2014-12-22 11:07:49 +0800
published: true
comments: true
categories: [golang, sublime, engineered]
tags: [golang, sublime, engineered]
keywords: go, golang, sublime,use sublime develop golang
description: 使用Sublime开发Golang

---

![golang logo](./images/blog/golang.jpg)

## 安装Golang

在[官网](http://golang.org)上直接下载安装包就可以了。下载pkg格式的最新安装包 ，直接双击运行，一路按照提示操作就可以完成安装。

或者使用brew进行安装```brew install go```

完成安装之后，打开终端，输入```go version```，检查golang sdk是否安装成功。

```go
➜  ~  go version
go version go1.3.3 darwin/amd64
```

## 环境变量配置

GOPATH是用来告诉Golang命令和其他相关工具 ，在哪儿可以找到你的Go包目录。

GOPATH是一个路径列表，类似PATH的配置。

```bash
GOPATH=~/workspace/demo/go-demo
```
我将Golang的相关配置都写在一个独立的.golangrc的文件中，然后在.zshrc中引入

```bash
source ~/.golangrc
```

> 如果你在GOLANG中配置了多个目录的话，当你下载开源包是(```go get *****```)，开源包默认会找到第一个目录，会统一下到第一个目录的pkg目录下。

## go官方推荐的项目结构

```bash
bin/
    hello                          # command executable
    outyet                         # command executable
pkg/
    linux_amd64/
        github.com/golang/example/
            stringutil.a           # package object
src/
    github.com/golang/example/
        .git/                      # Git repository metadata
	hello/
	    hello.go               # command source
	outyet/
	    main.go                # command source
	    main_test.go           # test source
	stringutil/
	    reverse.go             # package source
	    reverse_test.go        # test source
```

参考
> [http://golang.org/doc/code.html](http://golang.org/doc/code.html)

## Sublime安装GoSublime

Package Control如何安装我就不说了。直接安装[GoSublime](https://github.com/DisposaBoy/GoSublime)插件

Mac OSX下使用Command + Shift + P打开```Package Control```，然后输入 ```pcip``` (Package Control:Install Package的缩写)

在随后的界面中输入```GoSublime```，回车，就开始安装GoSubmlime了。

当你可以在Sublime的 ```Preferences -> Package Settings``` 看到 ```GoSublime``` ，那么恭喜你，你的基于Sublime的Golang开发环境就搭建完成了。
