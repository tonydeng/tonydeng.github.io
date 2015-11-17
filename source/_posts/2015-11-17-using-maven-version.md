layout: post
title: 在Java中使用Maven配置的版本信息
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [java, maven, properties, version]
date: 2015-11-17 18:56:48
keywords: java, maven, properties, version
---

## 场景描述

我们在开发一些项目的时候需要知道当前的版本状态，比如传递版本客户端信息到服务端等等。

那我们有些什么办法能够获取当前项目版本呢？

比较简单的办法就是在我们的程序中写一个常量来记录版本号，每次升级了就更新这个常量。但是这个方案还需要我们每次升级的时候都要记得这个事情，这个对于我这种记性不太好的人来说，简直就是灾难。

那还有什么更好的办法吗？

<!-- more -->

那我们可以从本身项目所使用的项目生命周期管理工具来考虑。比如现在`Java`的项目大部分还是用Maven来做项目生命周期管理的，那我们可不可以将`Maven`所记录的版本信息读取出来呢？

应该是可以，不过，如果要读取出来，我们是否需要去分析`Maven`的`pom.xml`文件吗？如果是这样，工作量也有点大。


其实`maven`已经给出了方案，就是可以在自定义的资源文件（`properties`）中放置`pom.xml`设置的变量，就会自动将变量替换成为真实地值。


## 例子：

我有一个hsc的项目，代码中需要知道当前项目的版本。那我应该怎么来做呢？

### 在src/main/resources/hsc-application.properties文件中放置如下内容：

```
hsc.version=${project.version}
```

### 配置pom.xml

```
	<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
            </resource>
        </resources>
	</build>
```

### 实现对hsc.version的读取

```
	/**
     * 获取HSC版本信息
     * @return
     */
    public static String getHSCVersion() {
        if (null == HSCVersion) {
            Properties properties = new Properties();
            try {
                properties.load(HSCUtils.class.getClassLoader().getResourceAsStream("hsc-application.properties"));
                if (!properties.isEmpty()) {
                    HSCVersion = properties.getProperty("hsc.version");
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return HSCVersion;
    }
```

### 测试获取版本信息

```
 	@Test
    public void testApplicationVersion(){
        String version = HSCUtils.getHSCVersion();

        log.info("hsc version:'{}'",version);
    }
```

## 参考

[maven可以使用内部变量](http://books.sonatype.com/mvnref-book/reference/resource-filtering-sect-properties.html)