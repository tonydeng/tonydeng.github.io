---
layout: post
title: "Go项目的目录结构"
date: 2015-02-10 10:29:15 +0800
published: true
comments: true
categories: [程序设计]
tags: [golang]
keywords: golang project directory structure
---

![Golang](/images/blog/GoLangKulani.jpg)

项目结构如何来组织，一般的开发语言都没有在语言层面上做规定，基本上都是在项目生命周期管理工具上对项目结构来做规定。不过`Go`在这方面做了相应地规定，这样可以在`Go`的开发者中保持一致。

<!-- more -->

## 一个Go项目在GOPATH下，会有如下三个目录

```
./bin
./pkg
./src
```

其中，`bin`存放编译后的可执行文件；`pkg`存放编译后的包文件；`src`存放项目源文件。

一般，`bin`和`pkg`目录可以不创建，`go`命令会自动创建(如 `go install`)，只需要创建`src`目录即可。

对于`pkg`目录，曾有人问：“我把`Go`中的包放入`pkg`下面，怎么不行啊？“，他时直接将Go包得源文件放入`pkg`中，这显然是不对的。`pkg`中的文件是`Go`编译生成的，而不是手动放进去的。（一般文件后缀名为`.a`）。

对于`src`目录，存放源文件，`Go`中的源文件以包（`package`）的形式组织。通常，新建一个包就在`src`目录中新建一个文件夹。

## 举例说明

比如：我新建一个项目`test`，开始的目录结构如下：

```
test/
|-- src/
```

为了编译方便，我在其中加入了一个`install`文件，目录结构如下：

```
test/
|-- install
`-- src/
```

其中`install`的内容如下：

```bash
#!/usr/bin/env bash

if [ ! -f install ]; then
    echo 'install must be run within its container folder' 1>&2
    exit 1
fi

CURDIR=`pwd`
OLDGOPATH="$GOPATH"
export GOPATH="$CURDIR"

gofmt -w src

go install test

export GOPATH="$OLDGOPATH"

echo 'finished'
```

之所以加上这个`install`，而不用配置`GOPATH`（避免新增一个`GO`项目就要往`GOPATH`中添加一个路径）

接下来，增加一个包： `config`和`main`程序。目录结构如下：

```
test/
|-- install
`-- src/
    |-- config
    |   `-- config.go
    `-- test
        `-- main.go
```

注意，`config.go`中的`package`名称最好和目录`config`一致，而文件名可以随意。`main.go`表示**`main`包**，文件名建议为`main.go`。（注：不一致时，生成的`.a`文件和目录名一致，这样，在`import`时，应该是目录名，而引用包时，需要包名。例如：目录为`myconfig`，包名为`config`，则生成的静态包文件是`myconfig.a`，引用该包:`import "myconfg"`,使用包中成员：`config.LoadConfig()`）

`config.go`和`main.go`的代码如下：

### `config.go`代码：

```go
package config
func LoadConfig(){

}
```

### `main.go`代码：

```go
package main

import (
    "config"
    "fmt"
)

func main(){
    config.LoadConfig()
    fmt.Println("Hello,GO!")
}
```

接下来，在项目根目录执行`./install`

这时候的目录结构如下：

```
test
|-- bin
|   `-- test
|-- install
|-- pkg
|   `-- linux_amd64
|       `-- config.a
`-- src
    |-- config
    |   `-- config.go
    `-- test
        `-- main.go
```

其中`config.a`是包`config`编译后生成的；`bin/test`是生成的可执行的二进制文件。

这是可以执行：`bin/test`。 输出结果为"`Hello,GO!`"

## 补充说明

1. 包可以有多层目录，比如`net/http`包，表示源文件再`src/net/http`目录下，不过源文件中的包名是最后一个目录的名字，比如`http`。 而在`import`包是，必须完整的路径，比如: `import "net/http"`

1. 有时候会见到`local import`(不建议使用)，语法类似这样：

```go
import "./config"
```

当代码有这样的语句是，很多时候都会见到类似的错误：

```go
local import "./config" in non-local package
```

> 我所了解的这种导入方式的使用是：当写入一个简单地测试脚本，想要使用`go run`命令时，可以使用这种导入方式。

比如上面的例子，把`test/main.go`移到`src`目录中，`test`目录删除，修改`main.go`中的`import "config"`为`import "./config"`，然后可以在src目录下执行： `go run main.go`

可见，`local import`不依赖`$GOPATH`
