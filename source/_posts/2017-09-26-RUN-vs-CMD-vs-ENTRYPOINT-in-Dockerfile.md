layout: post
title: RUN vs CMD vs ENTRYPOINT in Dockerfile
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [docker, Dockerfile, run, cmd, entrypoint]
date: 2017-09-26 17:09:47
keywords: RUN vs CMD vs ENTRYPOINT in Dockerfile
---

![docker logo](https://cdn.kelu.org/blog/tags/docker.jpg)

`RUN`、`CMD` 和 `ENTIRYPOINT`这三个`Dockerfile`指令看起来都很类似，很容易搞混。我们来通过一些实践来详细讨论一下它们之间的差别。

简单的说：

1. `RUN`执行命令并创建新的镜像层，`RUN`经常用来安装Docker image中需要的软件。
2. `CMD`设置容器启动后默认执行的命令及其参数，但`CMD`能够被`docker run`后面跟着的命令行参数代替。
3. `ENTIRYPOINT`配置容器启动时运行的命令。

<!-- more -->

## Shell和Exec格式

我们可以用Shell和Exec两种方式指定`RUN`、`CMD`和`ENTIRYPOINT`要运行的命令，这两者在使用中有细微的区别。

### Shell格式

```bash
<instruction> <command>
```

例如：

```Dockerfile
RUN apt-get install python3
CMD echo "Hello World"
ENTRYPOINT echo "Hello World"
```
当指令执行是，shell格式底层会调用`/bin/sh -c <command>`。

例如下面的Dockerfile片段：

```Dockerfile
ENV name Tony Deng
ENTRYPOINT echo "Hello, $name"
```

> 注意环境变量`name`已经被值`Tony Deng`替换了

### Exec格式

```bash
<instruction> ["executable","param1","param2",...]
```

例如：

```Dockerfile
RUN ["apt-get","install","python3"]
CMD ["/bin/echo","Hello World"]
ENTRYPOINT ["/bin/echo","Hello World"]
```

当指令执行是，会直接调用`<command>`，不会被`shell`解析。

例如下面的Dockerfile片段：

```Dockerfile
ENV name Tony Deng
ENTRYPOINT ["/bin/echo","Hello, $name"]
```

运行容器将输出

```bash
Hello, $name
```

> 注意环境变量`name`没有被替换

如果希望使用环境变量，可以这样来修改

```Dockerfile
ENV name Tony Deng
ENTRYPOINT ["/bin/sh","-c","echo Hello, $name"]
```

这样，容器将会输出

```bash
Hello, Tony Deng
```

`CMD`和`ENTRYPOINT`推荐使用`Exec`格式，因为指令可读性更强，更容易理解，`RUN`则两种格式都可以。


## RUN

`RUN`指令通常用于安装应用和软件包。

 `RUN`在当前镜像的顶部执行命令，并创建新的镜像层。`Dockerfile`中常常包含多个`RUN`指令。

安装多个包的例子：

```Dockerfile
RUN apt-get update && apt-get install -y \
    bzr \
    cvs \
    git \
    mercuial \
    subversion
```

> 注意： `apt-get update` 和 `apt-get install`最好放在一个`RUN`指令中执行，这样能够保证每次安装都是最新的包。如果`apt-get install`在单独的`RUN`执行，则会使用`apt-get update`创建新的镜像层，而这一层可能是很久以前缓存的。

## CMD

CMD指令允许用户指定容器的默认执行的命令。

此命令会在容器启动且docker run没有指定其他命令时执行。

1. 如果docker run指定了其他命令，CMD指定的默认命令将被忽略。
2. 如果Dockerfile中有多个CMD指令，只有最后一个CMD生效。

CMD有三种格式：

1. `Exec`格式： `CMD ["executable","param1","param2",...]`这是`CMD`推荐的格式。
2. `CMD ["param1","param2"]`为`ENTRYPOINT`提供额外的参数，此时`ENTRYPOINT`必须使用`Exec`格式。
3. `Shell`格式： `CMD command param1 param2`

`Exec` 和 `Shell` 格式前面已经介绍过了。
第二种格式`CMD ["param1","param2"]`要与`Exec`格式的`ENTRYPOINT`指令配合使用，其用途是为`ENTRYPOINT`设置默认的参数。我们将在后面讨论`ENTRYPOINT`时举例说明。

下面看看 CMD 是如何工作的。Dockerfile 片段如下：

```Dockerfile
CMD echo "Hello world"
```


运行容器 `docker run -it [image]` 将输出：

```bash
Hello world
```


但当后面加上一个命令，比如 `docker run -it [image] /bin/bash`，`CMD` 会被忽略掉，命令 `bash` 将被执行：

```bash
root@10a32dc7d3d3:/#
```

## ENTRYPOINT

`ENTRYPOINT`指令可让容器以应用程序或者服务的形式运行。

`ENTRYPOINT`看上去与`CMD`很像，它们都可以指定要执行的命令及其参数。不同的地方在于 `ENTRYPOINT`不会被忽略，一定会被执行，即使运行`docker run`时指定了其他命令。

`ENTRYPOINT`有两种格式：

1. `Exec` 格式：`ENTRYPOINT ["executable", "param1", "param2"]` 这是 `ENTRYPOINT` 的推荐格式。
1. `Shell` 格式：`ENTRYPOINT command param1 param2`

在为 `ENTRYPOINT` 选择格式时必须小心，因为这两种格式的效果差别很大。

### Exec 格式

`ENTRYPOINT` 的 `Exec` 格式用于设置要执行的命令及其参数，同时可通过 `CMD` 提供额外的参数。

`ENTRYPOINT` 中的参数始终会被使用，而 `CMD` 的额外参数可以在容器启动时动态替换掉。

比如下面的 `Dockerfile` 片段：

```Dockerfile
ENTRYPOINT ["/bin/echo", "Hello"]  

CMD ["world"]
```

当容器通过 `docker run -it [image]` 启动时，输出为：

```bash
Hello world
```

而如果通过 `docker run -it [image] TonyDeng` 启动，则输出为：

```bash
Hello TonyDengq
```

### Shell 格式

`ENTRYPOINT` 的 `Shell` 格式会忽略任何 `CMD` 或 `docker run` 提供的参数。

## 最佳实践

使用 `RUN` 指令安装应用和软件包，构建镜像。

如果 `Docker` 镜像的用途是运行应用程序或服务，比如运行一个 `MySQL`，应该优先使用 `Exec` 格式的 `ENTRYPOINT` 指令。`CMD` 可为 `ENTRYPOINT` 提供额外的默认参数，同时可利用 `docker run` 命令行替换默认参数。

如果想为容器设置默认的启动命令，可使用 `CMD` 指令。用户可在 `docker run` 命令行中替换此默认命令。

## 参考

[Dockerfile: ENTRYPOINT vs CMD](https://www.ctl.io/developers/blog/post/dockerfile-entrypoint-vs-cmd/)
[Abhishek Tomar在Slideshare关于Docker的分享](https://www.slideshare.net/abhishtomar/docker-43811773)
