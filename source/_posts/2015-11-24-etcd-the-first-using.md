layout: post
title: 初试ETCD
published: true
comments: true
sharing: true
footer: true
categories: [架构设计]
tags: [etcd, restful, api, 架构设计]
date: 2015-11-24 17:11:55
keywords: 架构设计 etcd 初试ETCD
---

![etcd logo](/images/blog/etcd/etcd-api.png)

之前我们分享过[ETCD应用场景](/2015/10/19/etcd-application-scenarios/)，所有的应用场景都需要etcd提供的api来做支撑，所以这次我们就来看看ectd提供的REST API如何来使用。

etcd 2.0之后，规范了端口号的使用，并且写入了[IANA组织的标准端口记录](http://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xml)。etcd将提供给外部客户端的端口变为2379，而etcd服务间通信的端口变为2380（不过现在依然还是兼容原来4001和7001端口）。
<!-- more -->

# 安装etcd

在mac osx下安装依然是非常简单，直接使用下面的命令就可以搞定。

```
➜  ~  brew install etcd
==> Downloading https://homebrew.bintray.com/bottles/etcd-2.2.1.el_capitan.bottle.tar.gz
Already downloaded: /Library/Caches/Homebrew/etcd-2.2.1.el_capitan.bottle.tar.gz
==> Pouring etcd-2.2.1.el_capitan.bottle.tar.gz
==> Caveats
To have launchd start etcd at login:
  ln -sfv /usr/local/opt/etcd/*.plist ~/Library/LaunchAgents
Then to load etcd now:
  launchctl load ~/Library/LaunchAgents/homebrew.mxcl.etcd.plist
==> Summary
🍺  /usr/local/Cellar/etcd/2.2.1: 7 files, 39M
```

我们来测试一下是否安装成功。

```
➜  ~  etcd -version
etcd Version: 2.2.1
Git SHA: GitNotFound
Go Version: go1.5.1
Go OS/Arch: darwin/amd64
```

OK，看到上面的信息，就说明你已经安装成功了。从上面的信息可以看到安装的etcd版本是2.2.1，go的版本是1.5.1。

# 启动etcd并测试服务

```
➜  ~  http GET http://127.0.0.1:2379/version
HTTP/1.1 200 OK
Content-Length: 44
Content-Type: application/json
Date: Wed, 25 Nov 2015 06:07:14 GMT

{
    "etcdcluster": "2.2.0",
    "etcdserver": "2.2.1"
}  
```

关于etcd启动参数及更多选项可以参考[Etcd Configuration Flags](https://github.com/coreos/etcd/blob/master/Documentation/configuration.md)

## 比较重要的配置

`-name` 节点名称，默认是UUID
`-data-dir` 保存日志和快照的目录，默认为当前工作目录
`-addr` 公布的ip地址和端口。 默认为127.0.0.1:2379
`-bind-addr` 用于客户端连接的监听地址，默认为-addr配置
`-peers` 集群成员逗号分隔的列表，例如 127.0.0.1:2380,127.0.0.1:2381
`-peer-addr` 集群服务通讯的公布的IP地址，默认为 127.0.0.1:2380.
`-peer-bind-addr` 集群服务通讯的监听地址，默认为-peer-addr配置

上述配置也可以设置配置文件，默认为`/etc/etcd/etcd.conf`。

# 试用etcd

## ectdctl介绍

我们可以使用[etcdctl](https://github.com/coreos/etcd/tree/master/etcdctl)这个官方提供的客户端来对etcd进行操作，可以从[github.com/coreos/etcd/releases](https://github.com/coreos/etcd/releases)下载。

etcdctl是一个命令行的客户端，它提供了一下简洁的命令，可以方便我们在对服务进行测试或者手动修改数据库内容。建议刚刚接触etcd的同学可以先通过cetdctl来熟悉相关操作。这些操作跟[HTTP API](https://github.com/coreos/etcd/blob/master/Documentation/api.md)基本上是对应的。


etcdctl支持下面列出来的命令，基本上可以分为数据库操作和非数据库操作，可以查看[etcdctl README.md](https://github.com/coreos/etcd/blob/master/etcdctl/README.md)来了解更多

```
➜  ~  etcdctl -h
NAME:
   etcdctl - A simple command line client for etcd.

USAGE:
   etcdctl [global options] command [command options] [arguments...]

VERSION:
   2.2.1

COMMANDS:
   backup		backup an etcd directory
   cluster-health	check the health of the etcd cluster
   mk			make a new key with a given value
   mkdir		make a new directory
   rm			remove a key or a directory
   rmdir		removes the key if it is an empty directory or a key-value pair
   get			retrieve the value of a key
   ls			retrieve a directory
   set			set the value of a key
   setdir		create a new or existing directory
   update		update an existing key with a given value
   updatedir		update an existing directory
   watch		watch a key for changes
   exec-watch		watch a key for changes and exec an executable
   member		member add, remove and list subcommands
   import		import a snapshot to a cluster
   user			user add, grant and revoke subcommands
   role			role add, grant and revoke subcommands
   auth			overall auth controls
   help, h		Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug			output cURL commands which can be used to reproduce the request
   --no-sync			don't synchronize cluster information before sending request
   --output, -o 'simple'	output response in the given format (`simple`, `extended` or `json`)
   --discovery-srv, -D 		domain name to query for SRV records describing cluster endpoints
   --peers, -C 			a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:4001,http://127.0.0.1:2379")
   --endpoint 			a comma-delimited list of machine addresses in the cluster (default: "http://127.0.0.1:4001,http://127.0.0.1:2379")
   --cert-file 			identify HTTPS client using this SSL certificate file
   --key-file 			identify HTTPS client using this SSL key file
   --ca-file 			verify certificates of HTTPS-enabled servers using this CA bundle
   --username, -u 		provide username[:password] and prompt if password is not supplied.
   --timeout '1s'		connection timeout per request
   --total-timeout '5s'		timeout for the command execution (except watch)
   --help, -h			show help
   --version, -v		print the version
```


## 数据库操作

数据库操作围绕对键值和目录的 `CRUD` （符合 REST 风格的一套操作：`Create`）完整生命周期的管理。

etcd 在键的组织上采用了层次化的空间结构（类似于文件系统中目录的概念），用户指定的键可以为单独的名字，如 `testkey`，此时实际上放在根目录 `/` 下面，也可以为指定目录结构，如 `cluster1/node2/testkey`，则将创建相应的目录结构。

> 注：CRUD 即 Create, Read, Update, Delete，是符合 REST 风格的一套 API 操作。


### set

指定某个键的值。例如

```
➜  ~  etcdctl set /testdir/testkey "Hello world"
Hello world
```

支持的选项包括：

```
--ttl '0'            该键值的超时时间（单位为秒），不配置（默认为 0）则永不超时
--swap-with-value value 若该键现在的值是 value，则进行设置操作
--swap-with-index '0'    若该键现在的索引值是指定索引，则进行设置操作
```

### get

获取指定键的值。例如

```
➜  ~  etcdctl get /testdir/testkey
Hello world
```

当键不存在时，则会报错。例如

```
➜  ~  etcdctl get /testdir/testkey2
Error:  100: Key not found (/testdir/testkey2) [18]
```

支持的选项为

```
--sort    对结果进行排序
--consistent 将请求发给主节点，保证获取内容的一致性
```

### update

当键存在时，更新值内容。例如

```
➜  ~  etcdctl update /testdir/testkey "Hello"
Hello
```
当键不存在时，则会报错。例如

```
➜  ~  etcdctl update /testdir/testkey2 "Hello"
Error:  100: Key not found (/testdir/testkey2) [19]
```

支持的选项为

```
--ttl '0'    超时时间（单位为秒），不配置（默认为 0）则永不超时
```

### rm

删除某个键值。例如

```
➜  ~  etcdctl rm /testdir/testkey
PrevNode.Value: Hello
```

当键不存在时，则会报错。例如

```
➜  ~  etcdctl rm /testdir/testkey
Error:  100: Key not found (/testdir/testkey) [20]
```

支持的选项为

```
--dir        如果键是个空目录或者键值对则删除
--recursive        删除目录和所有子键
--with-value     检查现有的值是否匹配
--with-index '0'    检查现有的 index 是否匹配
```

### mk

如果给定的键不存在，则创建一个新的键值。例如

```
➜  ~  etcdctl mk /testdir/testkey "Hello world"
Hello world
```

当键存在的时候，执行该命令会报错，例如

```
➜  ~  etcdctl mk /testdir/testkey "Hello world"
Error:  105: Key already exists (/testdir/testkey) [21]
```

支持的选项为

```
--ttl '0'    超时时间（单位为秒），不配置（默认为 0）则永不超时
```

### mkdir

如果给定的键目录不存在，则创建一个新的键目录。例如

```
➜  ~  etcdctl mkdir testdir2
```

当键目录存在的时候，执行该命令会报错，例如

```
➜  ~  etcdctl mkdir testdir2
Error:  105: Key already exists (/testdir2) [22]
```

支持的选项为

```
--ttl '0'    超时时间（单位为秒），不配置（默认为 0）则永不超时
```

### setdir

创建一个键目录，无论存在与否。

支持的选项为

```
--ttl '0'    超时时间（单位为秒），不配置（默认为 0）则永不超时
```

### updatedir

更新一个已经存在的目录。 支持的选项为

```
--ttl '0'    超时时间（单位为秒），不配置（默认为 0）则永不超时
```

### rmdir

删除一个空目录，或者键值对。

```
➜  ~  etcdctl setdir dir1
➜  ~  etcdctl rmdir dir1
```

若目录不空，会报错

```
➜  ~  etcdctl set /dir/testkey hi
hi
➜  ~  etcdctl rmdir /dir
Error:  108: Directory not empty (/dir) [29]
```

### ls

列出目录（默认为根目录）下的键或者子目录，默认不显示子目录中内容。

例如

```
➜  ~  etcdctl ls
/testdir
/testdir2
/dir
➜  ~  etcdctl ls dir
/dir/testkey
```

支持的选项包括

```
--sort    将输出结果排序
--recursive    如果目录下有子目录，则递归输出其中的内容
-p        对于输出为目录，在最后添加 `/` 进行区分
```

## 非数据库操作

### backup

备份 etcd 的数据。

支持的选项包括

```
--data-dir         etcd 的数据目录
--backup-dir     备份到指定路径
```

### watch

监测一个键值的变化，一旦键值发生更新，就会输出最新的值并退出。

例如，用户更新 testkey 键值为 Hello watch。


```
➜  ~  etcdctl get /testdir/testkey
Hello world
➜  ~  etcdctl set /testdir/testkey "Hello watch"
Hello watch
```

```
➜  ~  etcdctl watch testdir/testkey
Hello watch
```

支持的选项包括

```
--forever        一直监测，直到用户按 `CTRL+C` 退出
--after-index '0'    在指定 index 之前一直监测
--recursive        返回所有的键值和子键值
```

### exec-watch

监测一个键值的变化，一旦键值发生更新，就执行给定命令。

例如，用户更新 testkey 键值。

```
➜  ~  etcdctl exec-watch testkey -- sh -c 'ls'
default.etcd
Documentation
etcd
etcdctl
etcd-migrate
README-etcdctl.md
README.md
```

支持的选项包括

```
--after-index '0'    在指定 index 之前一直监测
--recursive        返回所有的键值和子键值
```

### member

通过 list、add、remove 命令列出、添加、删除 etcd 实例到 etcd 集群中。

例如本地启动一个 etcd 服务实例后，可以用如下命令进行查看。

```
$ etcdctl member list
ce2a822cea30bfca: name=default peerURLs=http://localhost:2380,http://localhost:7001 clientURLs=http://localhost:2379,http://localhost:4001
```
命令选项

```
--debug 输出 cURL 命令，显示执行命令的时候发起的请求
--no-sync 发出请求之前不同步集群信息
--output, -o 'simple' 输出内容的格式 (simple 为原始信息，json 为进行json格式解码，易读性好一些)
--peers, -C 指定集群中的同伴信息，用逗号隔开 (默认为: "127.0.0.1:4001")
--cert-file HTTPS 下客户端使用的 SSL 证书文件
--key-file HTTPS 下客户端使用的 SSL 密钥文件
--ca-file 服务端使用 HTTPS 时，使用 CA 文件进行验证
--help, -h 显示帮助命令信息
--version, -v 打印版本信息
```