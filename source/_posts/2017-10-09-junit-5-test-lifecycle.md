layout: post
title: JUnit5测试生命周期
published: true
comments: true
sharing: true
footer: true
categories: [JUnit5 Tutorial]
tags: [junit5,junit,tutorial,unit testing, test lifecycle]
date: 2017-10-09 16:57:29
keywords: JUnit5教程 生命周期
---

![junit5 logo](http://junit.org/junit4/images/junit5-banner.png)

在JUnit5中，测试生命周期由4个主要注解驱动，即`@BeforeAll`、`@BeforeEach`、`@AfterEach`和`@AfterAll`。与此同时，每个测试方法都必须标注`@Test`注解。

<!--more-->

## 生命周期结构

![junit 5 test lifecycle](http://junit.org/junit5/docs/current/user-guide/images/extensions_lifecycle.png)

## Before & After

在junit测试生命周期中，我们需要一些方法来设置和清除测试运行的环境或测试数据。

在JUnit中，对于每个测试 - 创建了一个新的测试实例。`@BeforeAll`和@`AfterAll`注释 - 以其名称清除 - 在整个测试执行周期中只应调用一次。所以他们必须被宣布

在JUnit中，对于每个测试 ，都会创建了一个新的测试实例。`@BeforeAll`和`@AfterAll`注解，在整个测试执行周期中只应调用一次。所以他们必须被声明为`static`。

如果他们是用相同注释注释的多个方法（例如两个方法`@BeforeAll`），那么它们的执行顺序是不确定的。

`@BeforeEach`和`@AfterEach`为每个测试实例调用，所以他们不需要`static`。

```java
public class AppTest {

    @BeforeAll
    static void setup(){
        System.out.println("@BeforeAll executed");
    }

    @BeforeEach
    void setupThis(){
        System.out.println("@BeforeEach executed");
    }

    @Test
    void testCalcOne()
    {
        System.out.println("======TEST ONE EXECUTED=======");
        Assertions.assertEquals( 4 , Calculator.add(2, 2));
    }

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

控制台输出

```bash
@BeforeAll executed

@BeforeEach executed
======TEST ONE EXECUTED=======
@AfterEach executed

@BeforeEach executed
======TEST TWO EXECUTED=======
@AfterEach executed

@AfterAll executed
```

## 禁用测试

要在JUnit 5中禁用测试，您将需要使用`@Disabled`注释。它相当于JUnit 4的`@Ignored`注释。

`@Disabled` 注释可以应用于测试类（禁用该类中的所有测试方法）或单独的测试方法。

```java
@Disabled
@Test
void testCalcTwo()
{
    System.out.println("======TEST TWO EXECUTED=======");
    Assertions.assertEquals( 6 , Calculator.add(2, 4));
}
```

## 断言

在任何测试方法中，ou将需要确定它是否通过失败。你可以使用断言来做。资产有助于通过测试用例的实际输出验证预期输出。为了保持简单，所有JUnit Jupiter断言是[org.junit.jupiter.Assertions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assertions.html)类中的静态方法。

```java
void testCase()
{
    //Test will pass
    Assertions.assertEquals(4, Calculator.add(2, 2));

    //Test will fail
    Assertions.assertEquals(3, Calculator.add(2, 2), "Calculator.add(2, 2) test failed");

    //Test will fail
    Supplier<String> messageSupplier  = ()-> "Calculator.add(2, 2) test failed";
    Assertions.assertEquals(3, Calculator.add(2, 2), messageSupplier);
}
```

要测试失败，只需使用`Assertions.fail()`方法。

```java
@Test
void testCase() {

    Assertions.fail("not found good reason to pass");
}
```

## 假设

[Assumptions](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Assumptions.html)提供了基于假设支持条件测试执行的静态方法。失败的假设导致测试被中止。无论何时继续执行给定的测试方法没有意义，通常使用假设。在测试报告中，这些测试将被标记为已通过。

假设类有两个方法：`assumeFalse()`，`assumeTrue()`。第三种方法`assumeThat()`处于实验状态，可能会在将来的版本中得到证实。

```java
@Test
void testOnDev()
{
    System.setProperty("ENV", "DEV");
    Assumptions.assumeTrue("DEV".equals(System.getProperty("ENV")));
    //remainder of test will proceed
}

@Test
void testOnProd()
{
    System.setProperty("ENV", "PROD");
    Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    //remainder of test will be aborted
}
```

## 更改默认测试实例生命周期

如果没有注释测试类或测试接口`@TestInstance`，则JUnit Jupiter将使用默认生命周期模式。

> 在jupiter api的TestInstance中定义的Lifecycle枚举

```java
package org.junit.jupiter.api;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.apiguardian.api.API;
import org.apiguardian.api.API.Status;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@API(
    status = Status.STABLE,
    since = "5.0"
)
public @interface TestInstance {
    TestInstance.Lifecycle value();

    public static enum Lifecycle {
        PER_CLASS,
        PER_METHOD;

        private Lifecycle() {
        }
    }
}
```

标准默认模式是`PER_METHOD`，但是，可以更改执行整个测试计划的默认值。

要更改默认的测试实例生命周期模式，只需将`junit.jupiter.testinstance.lifecycle.default` 配置参数设置为 `TestInstance.Lifecycle`忽略大小写中定义的枚举常量的名称。

这可以被提供作为一个JVM系统属性，作为配置参数在 LauncherDiscoveryRequest被传递到Launcher，或通过JUnit的平台配置文件。

例如，要设置成默认的测试生命周期模式`Lifecycle.PER_CLASS`,可以使用以下系统属性启动JVM。

> -Djunit.jupiter.testinstance.lifecycle.default=per_class

但是请注意，通过JUnit平台配置文件设置默认测试实例生命周期模式是一个更强大的解决方案，因为配置文件可以与项目一起检入版本控制系统，因此可以在IDE和您的构建软件中使用。

要`Lifecycle.PER_CLASS`通过JUnit平台配置文件设置默认测试实例生命周期模式，请创建一个以`junit-platform.properties`类路径根目录命名的文件（例如，`src/test/resources`），具有以下内容。

```java
junit.jupiter.testinstance.lifecycle.default = per_class
```

> 更改默认的测试实例生命周期模式可能导致不可预测的结果和脆弱的构建。
