layout: post
title: Java8环境下的Maven javadoc插件的配置
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [java,java8, maven, javadoc,javadoc-plugin,maven-plugin,plugin]
date: 2015-10-21 20:45:09
keywords: java java8 maven javadoc javadoc-plugin maven-plugin plugin
---
## 问题
今天用maven在release代码时，又出现新的问题了，生成javadoc出现异常，导致release失败。


```
Refer to the generated Javadoc files in './target/site/apidocs' dir.

org.apache.maven.reporting.MavenReportException:
Exit code: 1 -./src/main/java/com/github/tonydeng/commons/utils/DigestUtils.java:30: 警告: input没有 @param
	public static String sha1ToHex(String input) {
	                     ^
./src/main/java/com/github/tonydeng/commons/utils/DigestUtils.java:30: 警告: 没有 @return
```
<!-- more -->
## 配置

javadoc的插件配置是这样

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.9.1</version>
        <executions>
            <execution>
                <id>attach-javadocs</id>
                <phase>install</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        <configuration>
    <encoding>UTF-8</encoding>
        <charset>UTF-8</charset>
    </configuration>
 </plugin>
```

## 排查

之前这个配置工作都很正常，让我百思不得其解。

那怎么办？

嗯，先升级一下maven-javadoc-plugin看看，升级到最新的2.10.3。

还是不行，回退java版本到java7。

哦，这样就可以了。看来应该是java8加了对javadoc的新的特性，我没有关注到。

OK，找到线索了，那就查查看。

先看看[Java8的特性列表](http://openjdk.java.net/projects/jdk8/features)吧。果然，Java8添加了一个Javadoc注释内容检查的特性[Doclint](http://openjdk.java.net/jeps/172)。

DocLint提供了一种方法来检测Javadoc的注释中的错误，希望能够在开发周期的早期和容易链接回源代码的方式。

## 解决

如果想忽略DocLint的使用，可以在maven-javadoc-plugin的配置中加上对DocLint的忽略。

最后的配置如下：

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.10.3</version>
        <executions>
            <execution>
                <id>attach-javadocs</id>
                <phase>install</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        <configuration>
        <encoding>UTF-8</encoding>
        <charset>UTF-8</charset>
        <additionalparam>-Xdoclint:none</additionalparam>
    </configuration>
 </plugin>
```
