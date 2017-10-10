layout: post
title: JUnit5教程-简介
published: true
comments: true
sharing: true
footer: true
categories: [JUnit5 Tutorial]
tags: [junit5,junit,tutorial,unit testing]
date: 2017-10-09 11:09:07
keywords: JUnit5教程 简介
---


![junit5 logo](http://junit.org/junit4/images/junit5-banner.png)


这个[JUnit5教程](/categories/JUnit5-Tutorial/)将讲述如何使用Java8风格的编码以及其他功能，同时也了解[JUnit5](http://junit.org/junit5/)与之前的版本的区别。

## Junit5简介

JUnit是Java中使用最广泛的测试框架，之前Java8发布了最引人注目的lambda表达式，整个Java的编码风格发生巨大的变化，JUnit5主要在希望能够适应Java8风格的编码以及相关工，这就是为什么建议在Java8之后的项目中使用JUnit5来创建和执行测试。

JUnit官方说明：

> JUnit 5 is the next generation of JUnit. The goal is to create an up-to-date foundation for developer-side testing on the JVM. This includes focusing on Java 8 and above, as well as enabling many different styles of testing.

JUnit5的第一个可用性版本是在2017年9月10日发布的。

<!-- more -->

## JUnit5架构

相比JUnit4，JUnit5由三个不同的子项目及不同的模块组成。

> JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

下图是JUnit的架构图

![junit 5 architecture ](https://www.ibm.com/developerworks/library/j-introducing-junit5-part1-jupiter-api/Figure-1.png)

### 1. JUnit Platform

启动Junit测试、IDE、构建工具或插件都需要包含和扩展Platform API，它定义了`TestEngine`在平台运行的新测试框架的API。

它还提供了一个控制台启动器，可以从命令行启动Platform，为Gradle和Maven插件提供支持。

### 2. JUnit Jupiter

它用于编写测试代码的新的编程和扩展模型。它具有所有新的Junit注释和`TestEngine`实现来运行这些注释编写的测试。

### 3. JUnit Vintage

它主要的目的是支持在JUnit5的测试代码中运行JUnit3和4方式写的测试，它能够向前兼容之前的测试代码。

下图是JUnit5的依赖关系图

![junit 5 dependence](http://junit.org/junit5/docs/current/user-guide/images/component-diagram.svg)

## 安装

你可以在Maven或Gradle项目中使用JUnit5，包含最小的两个依赖关系，即`junit-jupiter-engince`和`junit-platform-runner`。

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>${junit.jupiter.version}</version>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-runner</artifactId>
    <version>${junit.platform.version}</version>
    <scope>test</scope>
</dependency>
```

```gradle
testRuntime("org.junit.jupiter:junit-jupiter-engine:5.0.0-M4")
testRuntime("org.junit.platform:junit-platform-runner:1.0.0-M4")
```

如果你想了解更多的话，可以查看官方的[build.gradle](https://github.com/junit-team/junit5-samples/blob/master/junit5-gradle-consumer/build.gradle)和[pom.xml](https://github.com/junit-team/junit5-samples/blob/master/junit5-maven-consumer/pom.xml)。

## JUnit5 Annotations

JUnit5提供了以下的注解来编写测试代码。

|Annotations|描述|
|--|--|
|`@BeforeEach`|在方法上注解，在每个测试方法运行之前执行。|
|`@AfterEach`|在方法上注解，在每个测试方法运行之后执行|
|`@BeforeAll`|该注解方法会在所有测试方法之前运行，该方法必须是静态的。|
|`@AfterAll`|该注解方法会在所有测试方法之后运行，该方法必须是静态的。|
|`@Test`|用于将方法标记为测试方法|
|`@DisplayName`|用于为测试类或测试方法提供任何自定义显示名称|
|`@Disable`|用于禁用或忽略测试类或方法|
|`@Nested`|用于创建嵌套测试类|
|`@Tag`|用于测试发现或过滤的标签来标记测试方法或类|
|`@TestFactory`|标记一种方法是动态测试的测试工场|

> 可以查看详细的[JUnit5注解](/2017/10/10/junit-5-annotations/)说明

## 使用JUnit5编写测试

JUnit4和JUnit5在测试编码风格上没有太大变化。这是其[生命周期](/2017/10/09/junit-5-test-lifecycle/)方法的样本测试。

```java
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

import com.github.tonydeng.junit5.examples.Calculator;

public class AppTest {

    @BeforeAll
    static void setup(){
        System.out.println("@BeforeAll executed");
    }

    @BeforeEach
    void setupThis(){
        System.out.println("@BeforeEach executed");
    }

    @Tag("DEV")
    @Test
    void testCalcOne()
    {
        System.out.println("======TEST ONE EXECUTED=======");
        Assertions.assertEquals( 4 , Calculator.add(2, 2));
    }

    @Tag("PROD")
    @Disabled
    @Test
    void testCalcTwo()
    {
        System.out.println("======TEST TWO EXECUTED=======");
        Assertions.assertEquals( 6 , Calculator.add(2, 4));
    }

    @AfterEach
    void tearThis(){
        System.out.println("@AfterEach executed");
    }

    @AfterAll
    static void tear(){
        System.out.println("@AfterAll executed");
    }
}
```

## 测试套件

使用JUnit5的测试套件，可以将测试扩展到多个测试类和不同的软件包。JUnit5提供了两个注解： [@SelectPackages](http://junit.org/junit5/docs/current/api/index.html?org/junit/platform/runner/SelectPackages.html) 和 [@SelectClasses](http://junit.org/junit5/docs/current/api/index.html?org/junit/platform/runner/SelectClasses.html) 来创建测试套件。

要执行测试套件，可以是使用 `@RunWith(JUnitPlatform.class)`

```Java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
public class JUnit5TestSuiteExample
{
}
```

另外，你也可以使用以下注解来过滤测试包、类甚至测试方法。

1. `@IncludePackages`和`@ExcludePackages`来过滤包
2. `@IncludeClassNamePatterns`和`@ExcludeClassNamePatterns`过滤测试类
3. `@IncludeTags`和`@ExcludeTags`过滤测试方法

```java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@IncludePackages("com.github.tonydeng.junit5.examples.packageC")
@ExcludeTags("PROD")
public class JUnit5TestSuiteExample
{
}
```

## 断言

断言有助于使用测试用例的实际输出验证预期输出。为了保持简单，所有JUnit Jupiter断言是[org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html)类中的静态方法，例如`assertEquals()`，`assertNotEquals()`。

```java
void testCase()
{
    //Test will pass
    Assertions.assertNotEquals(3, Calculator.add(2, 2));

    //Test will fail
    Assertions.assertNotEquals(4, Calculator.add(2, 2), "Calculator.add(2, 2) test failed");

    //Test will fail
    Supplier<String> messageSupplier  = ()-> "Calculator.add(2, 2) test failed";
    Assertions.assertNotEquals(4, Calculator.add(2, 2), messageSupplier);
}
```

## 假设

[Assumptions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assumptions.html)类提供了静态方法来支持基于假设的条件测试执行。失败的假设导致测试被中止。无论何时继续执行给定的测试方法没有意义，通常使用假设。在测试报告中，这些测试将被标记为已通过。

JUnit的Jupiter假设类有两个这样的方法：`assumeFalse()`，`assumeTrue()`。

```java
public class AppTest {
    @Test
    void testOnDev()
    {
        System.setProperty("ENV", "DEV");
        Assumptions.assumeTrue("DEV".equals(System.getProperty("ENV")), AppTest::message);
    }

    @Test
    void testOnProd()
    {
        System.setProperty("ENV", "PROD");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }

    private static String message () {
        return "TEST Execution Failed :: ";
    }
}
```

## JUnit3或4的兼容性

JUnit4已经存在了很长时间，并且有许多以JUnit4编写的测试.JUnit Jupiter还需要支持这些测试。为此，开发了JUnit Vintage子项目。

JUnit Vintage提供了TestEngine在JUnit 5平台上运行基于JUnit 3和JUnit 4的测试的实现。
