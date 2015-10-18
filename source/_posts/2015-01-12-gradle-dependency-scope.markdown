---
layout: post
title: "Gradle依赖范围介绍"
date: 2015-01-12 13:47:19 +0800
published: true
comments: true
categories: [效率]
tags: [gradle,engineered]
keywords:  gradle dependency
description: gradle依赖范围介绍
---

![Gradle Logo](/images/blog/gradle.png)

自从Google推出Android的集成开发环境（IDE）-- [Android Studio](http://developer.android.com/tools/studio/index.html)，默认集成了[Gradle](http://www.gradle.org)来进行对Android项目生命周期的管理。那我们也需要从原来的Ant转成Gradle。

对于开发工程师来说，像Gradle这样的工具，第一体验是对依赖的管理。团队里面有不少原来是使用[Maven](http://maven.apache.org)的同学，会有这样的问题：“maven的依赖管理除了最基本的坐标体系（groupId、artifactId、version、packaging）以外，还有一个scope的概念。那作为继承了maven的依赖体系的gradle，它的依赖范围又有哪些？”


### 那我们来看看Gradle的依赖范围（dependent scope到底有哪些）。

#### war插件使用的依赖范围

* providedCompile

        war插件提供的范围类型:与compile作用类似,但不会被添加到最终的war包中这是由于编译、测试阶段代码需要依赖此类jar包，而运行阶段容器已经提供了相应的支持，所以无需将这些文件打入到war包中了;例如Servlet API就是一个很明显的例子.

* providedRuntime

        同providedCompile类似，也是war插件提供的范围类型。它的范围和jar插件的runtime基本一样，。

相关说明可以看看Gradle官方文档的[第26章第4节 The War Plugin](http://www.gradle.org/docs/current/userguide/war_plugin.html#N131A1)

#### java插件使用的依赖范围

* compile

        编译范围依赖在所有的classpath中可用，同时它们也会被打包。

* runtime

        runtime依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现。

* testCompile

        测试期编译需要的附加依赖

* testRuntime

        测试期编译需要的附加依赖

* archives

        项目构件时会使用的依赖范围

* default

        配置默认依赖范围，用在这个项目上一个项目依赖的默认配置。包含本项目在运行时所需的构件和依赖。

相关说明可以查看Gradle官方文档的[第23章第5节 The Java Plugin](http://www.gradle.org/docs/current/userguide/java_plugin.html#sec:java_plugin_and_dependency_management)


其他插件的依赖范围基本上也就是这两个插件的子集了，了解了上述的依赖范围也就等于了解Gradle所有的依赖范围了。


### Gradle的依赖范围与Tasks之间的关系

![依赖范围与Gradle Tasks之间的关系](/images/blog/javaPluginConfigurations.png)
