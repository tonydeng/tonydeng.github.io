layout: post
title: 安装Leiningen
published: true
comments: true
sharing: true
footer: true
categories: [Leiningen + Clojure之旅]
tags: [leiningen, clojure]
date: 2015-11-19 17:56:48
keywords:
---

![leiningen logo](/images/blog/leiningen.png)

## Leiningen简单介绍

[Leiningen](http://leiningen.org)是[Clojure](https://clojure.org)（貌似需要自备梯子）的项目生命周期管理工具，就像[Maven](http://maven.apache.org)在[Java](https://java.com)中的地位一样。

关于Leiningen具体的情况和使用方法，它的[官网](http://leiningen.org)和[GitHub](https://github.com/technomancy/leiningen)看看，上面会有更清楚的描述。
<!--more-->
## 安装过程

我只是给大家说说，在安装Leiningen时可能会碰到的坑。

先说说，基本安装流程，咱们就遇坑填坑。

### 下载lein命令

下载Leiningen很简单，你使用我的命令来直接下载（仅限Mac OSX和Linux）。当然，你也可以尝试用自己系统的包管理系统来安装。比如Mac OSX可以使用`brew install leiningen`。

```
$ cd /usr/local/bin
$ wget https://raw.githubusercontent.com/technomancy/leiningen/stable/bin/lein
$ chmod 755 lein
```

那我们是否就已经算是安装成功，可以使用Leiningen了呢？别着急，你要是有兴趣，你看看lein这个命令，它其实就只是一段`Shell`脚本，光有它还是不够的。

### 下载Leiningen

你可以执行一下lein，它就会自动帮你下载正在工作的那个具有超牛力的Leiningen。

```
$ lein 
Downloading Leiningen to /Users/tonydeng/.lein/self-installs/leiningen-2.5.3-standalone.jar now...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   605    0   605    0     0    262      0 --:--:--  0:00:02 --:--:--   262
  0 15.0M    0     0    0     0      0      0 --:--:--  0:00:37 --:--:--     0
  0 15.0M    0     0    0     0      0      0 --:--:--  0:00:38 --:--:--     0
```

然后我就满心期待的等啊等啊，可惜等到的只是一个让人沮丧的错误提示。

```
Failed to download https://github.com/technomancy/leiningen/releases/download/2.5.3/leiningen-2.5.3-standalone.zip (exit code 56)
It's possible your HTTP client's certificate store does not have the
correct certificate authority needed. This is often caused by an
out-of-date version of libssl. It's also possible that you're behind a
firewall and haven't set HTTP_PROXY and HTTPS_PROXY.
```

看到这个提示之后，我的第一反应就是 ~~Fuck GFW~~！赶紧搭上梯子，再试，依然是不好使。然后怀疑是`curl`的问题，升级`curl`到最新版本，依然是不行。没有办法，只好祭出Google（min）大（gan）法（ci）。

就看到了下面这个leiningen项目中的issue #1634 -- [Leiningen installation issue on Debian Wheezy #1634](https://github.com/technomancy/leiningen/issues/1634)，试了试上面给的方案，设置一下`HTTP_CLIENT`，毕竟上面的错误提示也说到了证书的问题。

```
export HTTP_CLIENT="wget --no-check-certificate -O"
```

这些就OK了。

不过，让我比较郁闷的，为什么Leiningen版本升级之后，反而错误提示比以前的版本要检阅了呢？还是说，我使用的系统和这个 `issue #1634`遇到问题的操作系统不一样呢？不管怎么样，也算是安装成功了。

当你看到下面的命令输出，就说明安装成功了。

```
$ lein --version
Leiningen 2.5.3 on Java 1.8.0_40 Java HotSpot(TM) 64-Bit Server VM
```

你可以使用Leiningen的帮助来简单看看它提供的功能。

```
$ lein -h
Leiningen is a tool for working with Clojure projects.

Several tasks are available:
change              Rewrite project.clj by applying a function.
check               Check syntax and warn on reflection.
classpath           Print the classpath of the current project.
clean               Remove all files from project's target-path.
compile             Compile Clojure source into .class files.
deploy              Build and deploy jar to remote repository.
deps                Download all dependencies.
do                  Higher-order task to perform other tasks in succession.
help                Display a list of tasks or help for a given task.
install             Install the current project to the local repository.
jar                 Package up all the project's files into a jar file.
javac               Compile Java source files.
new                 Generate project scaffolding based on a template.
plugin              DEPRECATED. Please use the :user profile instead.
pom                 Write a pom.xml file to disk for Maven interoperability.
release             Perform :release-tasks.
repl                Start a repl session either with the current project or standalone.
retest              Run only the test namespaces which failed last time around.
run                 Run a -main function with optional command-line arguments.
search              Search remote maven repositories for matching jars.
show-profiles       List all available profiles or display one if given an argument.
test                Run the project's tests.
trampoline          Run a task without nesting the project's JVM inside Leiningen's.
uberjar             Package up the project files and dependencies into a jar file.
update-in           Perform arbitrary transformations on your project map.
upgrade             Upgrade Leiningen to specified version or latest stable.
vcs                 Interact with the version control system.
version             Print version for Leiningen and the current JVM.
with-profile        Apply the given task with the profile(s) specified.

Run `lein help $TASK` for details.

Global Options:
  -o             Run a task offline.
  -U             Run a task after forcing update of snapshots.
  -h, --help     Print this help or help for a specific task.
  -v, --version  Print Leiningen's version.

These aliases are available:
downgrade, expands to upgrade

See also: readme, faq, tutorial, news, sample, profiles, deploying, gpg,
mixed-source, templates, and copying.
```


接下来，我就要使用Leiningen来创建和管理Clojure项目了。可以参考一下[官方提供的简单project.clj例子](https://github.com/technomancy/leiningen/blob/stable/sample.project.clj)。

## 新建Clojure项目

用下面的命令就可以轻松创建一下用Leiningen管理的Clojure项目。

```
➜  lein new clojure-demo
```

我们来看看项目的目录结构，非常全面。

```
➜  clojure-demo git:(master)  tree
.
├── CHANGELOG.md
├── LICENSE
├── README.md
├── clojure-demo.iml
├── dev-resources
├── doc
│   └── intro.md
├── project.clj
├── resources
├── src
│   └── clojure_demo
│       └── core.clj
├── target
│   ├── classes
│   │   └── META-INF
│   │       └── maven
│   │           └── clojure-demo
│   │               └── clojure-demo
│   │                   └── pom.properties
│   └── stale
│       └── extract-native.dependencies
└── test
    └── clojure_demo
        └── core_test.clj
```
好了，今天就先弄到这儿。改天在继续更新这个Leiningen + Clojure之旅，比如Leiningen和Maven的整合、Clojure和Java的互相调用等等。