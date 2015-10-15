---
layout: post
title: "解决Maven使用artchetype:create创建工程失败的问题"
date: 2015-09-18 13:32:48 +0800
published: true
comments: true
categories: maven
keywords: maven
description: 解决Maven使用artchetype:create创建工程失败的问题
---

![maven java](/images/blog/maven/maven-jee.png)

Maven是我一直用来管理Java项目生命周期的工具，从2006年开始使用，到现在快十年了。没想到今天碰到一个新的问题，就是使用 `mvn artchetype:create` 来创建项目时失败了。

问题如下：

Maven命令如下：

```
mvn archetype:create \
    -DgroupId=com.github.tonydeng.bluebrid \
    -DartifactId=bluebrid
```

错误提示如下：

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-archetype-plugin:2.4:create (default-cli) on project bluebrid: Unable to parse configuration of mojo org.apache.maven.plugins:maven-archetype-plugin:2.4:create for parameter #: Cannot create instance of interface org.apache.maven.artifact.repository.ArtifactRepository: org.apache.maven.artifact.repository.ArtifactRepository.<init>() -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginConfigurationException
```

从错误提示来看，应该是`maven-archetype-plugin-2.4`这个版本的插件没有`create`这个参数了。


去[Maven的官网](http://maven.apache.org/)的[plugin list](http://maven.apache.org/plugins/index.html)中查看，原来archetype这个插件在2015年8月9号更新到2.4的版本，那再继续看看[archetype插件](http://maven.apache.org/archetype/maven-archetype-plugin/)有了什么新的变化。

原来artchetype已经没有create这个方法了，现在推荐方式是先建立自己的Example project，然后通过这个example project为例子，再创建实际你要用的项目。

基本的流程就像这幅图中描述的。

![maven archetype plugin flow](/images/blog/maven/archetype-overview.png)

OK，那我们就按照官方的指示来操作一次看看。

```
$ mvn archetype:create-from-project
$ cd target/generated-sources/archetype/
$ mvn install
$ cd /tmp/archetype
$ mvn archetype:generate -DarcheypteCatalog=local
```

那还有没有别的简单的方法来创建，就像之前使用archetype:create一样呢？


你可以使用下面的命令来进行创建项目。

```
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.maven.archetypes \
    -DgroupId=com.github.tonydeng.bluebrid \
    -DartifactId=bluebrid \
    -T20
```

不过，这样创建项目，你会发现在如下两个点要等N长的时间，这个N长的时间可能要以几十分钟来计算。

> [INFO] Generating project in Interactive mode

> [INFO] Generating project in Batch mode

其实你可以再加上两个参数  `-DinteractiveMode=false -DarchetypeCatalog=internal` 可以让你快速秒建项目。

这两个参数的意义分别如下：

> -DinteractiveMode=false  指定不使用交互模式
> 
> -DarchetypeCatalog=internal 指定不从远程服务器上取catalog，

完整的命令如下：

```
mvn archetype:generate \
    -DarchetypeGroupId=org.apache.maven.archetypes \
    -DinteractiveMode=false \
    -DarchetypeCatalog=internal \
    -DgroupId=com.github.tonydeng.bluebrid \
    -DartifactId=bluebrid \
    -T20
```

遇到同样问题的同学们，你们可以试试我的方法，应该能够快速的解决你们的问题。