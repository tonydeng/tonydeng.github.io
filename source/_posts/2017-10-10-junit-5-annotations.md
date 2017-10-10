layout: post
title: JUnit5教程-注解
published: true
comments: true
sharing: true
footer: true
categories: [JUnit5 Tutorial]
tags: [junit5,junit,tutorial,unit testing, annotations]
date: 2017-10-10 10:33:44
keywords: JUnit5教程 注解 junit5 annotations
---

![junit5 logo](http://junit.org/junit4/images/junit5-banner.png)


## JUnit5提供的注解

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


<!-- more -->

## JUnit5 VS JUnit4

![junit5 vs junit4](https://image.slidesharecdn.com/junit5vsjunit4-170324194457/95/junit-5-vs-junit-4-9-638.jpg)
> 本图来自[Ismael](https://www.slideshare.net/rkmael)在Slideshare分享的[JUnit 5 vs JUnit 4](https://www.slideshare.net/rkmael/junit-5-vs-junit-4)


接下来来我们来一个个注解详细介绍。

## @BeforeAll

如[JUnit5 VS JUnit4](#JUnit5 VS JUnit4)图中所示，`@BeforeAll`是替换JUnit4的`@BeforeClass`。它用于表示在当前测试类中的所有测试之前应该执行注解的方法。

### @BeforeAll使用

```java
@BeforeAll
public void init(){
    System.out.println("Before All init() method called");
}
```

> `@BeforeAll`注解方法必须是静态方法，否则会抛出运行时错误。

```java
org.junit.platform.commons.JUnitException: @BeforeAll method 'public void com.github.tonydeng.junit5.examples.JUnit5AnnotationsExample.init()' must be static.
at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.assertStatic(LifecycleMethodUtils.java:66)
at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.lambda$findBeforeAllMethods$0(LifecycleMethodUtils.java:42)
at java.util.ArrayList.forEach(ArrayList.java:1249)
at java.util.Collections$UnmodifiableCollection.forEach(Collections.java:1080)
at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.findBeforeAllMethods(LifecycleMethodUtils.java:42)
```

### @BeforeAll示例

我们来举个例子。我使用了一个`Calculator`类并添加了一个`add`方法。我将使用`@RepeatedTest`注解测试它5次。此注解将导致`add`测试运行5次。但`@BeforeAll`注解方法只能调用一次。

```java
package com.github.tonydeng.junit5.examples;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(5)
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @BeforeAll
    public static void init(){
        System.out.println("Before All init() method called");
    }
}
```

计算器类是：

```java
package com.github.tonydeng.junit5.examples;

public class Calculator
{
    public int add(int a, int b) {
        return a + b;
    }
}
```

现在执行测试，您将在控制台输出下方看到：

```bash
Before All init() method called
```

显然，@BeforeAll注解init()方法只调用一次。

## @BeforeEach

如[JUnit5 VS JUnit4](#JUnit5 VS JUnit4)图中所示，`@BeforeEach`注解是替换JUnit4中的`@Before`注解。它用于表示`@Test`在当前类中的每个方法之前应该执行注解方法。

### @BeforeEach用法


```java
@BeforeEach
public void initEach(){
    System.out.println("Before Each initEach() method called");
}
```

> @BeforeEach 注解方法不能是静态方法，否则会抛出运行时错误。

```java
org.junit.platform.commons.JUnitException: @BeforeEach method 'public static void com.github.tonydeng.junit5.examples.JUnit5AnnotationsExample.initEach()' must not be static.
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.assertNonStatic(LifecycleMethodUtils.java:73)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.lambda$findBeforeEachMethods$2(LifecycleMethodUtils.java:54)
    at java.util.ArrayList.forEach(ArrayList.java:1249)
    at java.util.Collections$UnmodifiableCollection.forEach(Collections.java:1080)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.findBeforeEachMethods(LifecycleMethodUtils.java:54)
```


### @BeforeEach示例

我还是使用之前`Calculator`类的add方法。我将使用`@RepeatedTest`注解测试它5次。此注解将导致`add`测试运行5次。对于每次运行的测试方法，`@BeforeEach`注解方法也应该每次运行。

```java
package com.github.tonydeng.junit5.examples;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(5)
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @BeforeAll
    public static void init(){
        System.out.println("Before All init() method called");
    }

    @BeforeEach
    public void initEach(){
        System.out.println("Before Each initEach() method called");
    }
}
```

现在执行测试，您将在控制台输出下方看到：

```bash
Before All init() method called
Before Each initEach() method called
Before Each initEach() method called
Before Each initEach() method called
Before Each initEach() method called
Before Each initEach() method called
Before Each initEach() method called
```
显然，每个测试方法调用调用一次`@BeforeEach`注解`initEach()`方法。

## @AfterEach


如[JUnit5 VS JUnit4](#JUnit5 VS JUnit4)图中所示，`@AfterJUnit`注解是替换JUnit4中的`@Before`注解。它用于表示`@Test`在当前类中的每个方法之前应该执行注解方法。

### @AfterEach注解用法


注解方法@AfterEach如下：
```java
@AfterEach
public void cleanUpEach(){
    System.out.println("After Each cleanUpEach() method called");
}
```

> @AfterEach 注解方法不能是静态方法，否则会抛出运行时错误。

```java
org.junit.platform.commons.JUnitException: @AfterEach method 'public static void com.github.tonydeng.junit5.examples.JUnit5AnnotationsExample.cleanUpEach()' must not be static.
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.assertNonStatic(LifecycleMethodUtils.java:73)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.lambda$findAfterEachMethods$3(LifecycleMethodUtils.java:60)
    at java.util.ArrayList.forEach(ArrayList.java:1249)
    at java.util.Collections$UnmodifiableCollection.forEach(Collections.java:1080)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.findAfterEachMethods(LifecycleMethodUtils.java:60)
```

### @AfterEach注解示例


我还是使用之前`Calculator`类的add方法。我将使用`@RepeatedTest`注解测试它5次。此注解将导致`add`测试运行5次。对于每次运行的测试方法，`@AfterEach`注解方法也应该每次运行。

```java
package com.github.tonydeng.junit5.examples;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(5)
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @AfterAll
    public static void cleanUp(){
        System.out.println("After All cleanUp() method called");
    }

    @AfterEach
    public void cleanUpEach(){
        System.out.println("After Each cleanUpEach() method called");
    }

}
```

现在执行测试，您将在控制台输出下方看到：

```bash
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After All cleanUp() method called
```

显然，每个测试方法调用调用一次`@AfterEach`注解`cleanUpEach()`方法。

## @AfterAll

如[JUnit5 VS JUnit4](#JUnit5 VS JUnit4)图中所示，`@AfterAll`注解是替换JUnit4中的`@AfterClass`注解。它用于表示在当前测试类中的所有测试后应执行注解方法。

### @AfterAll注解使用

注解方法@AfterAll如下：

```java
@AfterAll
public static void cleanUp(){
    System.out.println("After All cleanUp() method called");
}
```

>@AfterAll 注解方法必须是静态方法，否则会抛出运行时错误。

```java
org.junit.platform.commons.JUnitException: @AfterAll method 'public void com.github.tonydeng.junit5.examples.JUnit5AnnotationsExample.cleanUp()' must be static.
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.assertStatic(LifecycleMethodUtils.java:66)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.lambda$findAfterAllMethods$1(LifecycleMethodUtils.java:48)
    at java.util.ArrayList.forEach(ArrayList.java:1249)
    at java.util.Collections$UnmodifiableCollection.forEach(Collections.java:1080)
    at org.junit.jupiter.engine.descriptor.LifecycleMethodUtils.findAfterAllMethods(LifecycleMethodUtils.java:48)
```

### @AfterAll注解示例

我还是使用之前`Calculator`类的add方法。我将使用`@RepeatedTest`注解测试它5次。此注解将导致`add`测试运行5次。但`@AfterAll`注解方法只能调用一次。

```java
package com.github.tonydeng.junit5.examples;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(5)
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @AfterAll
    public static void cleanUp(){
        System.out.println("After All cleanUp() method called");
    }

    @AfterEach
    public void cleanUpEach(){
        System.out.println("After Each cleanUpEach() method called");
    }

}
```

现在执行测试，您将在控制台输出下方看到：

```bash
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After Each cleanUpEach() method called
After All cleanUp() method called
```
显然，`@AfterAll`注解`cleanUp()`方法只调用一次。

## @RepeatedTest

JUnit5 `@RepeatedTest`注解能够编写可重复的测试模板，可以多次运行。频率可以配置为`@RepeatedTest`注解的参数。

### @RepeatedTest注解用法

要创建可重复的测试模板方法，请使用注解方法@RepeatedTest。

```java
@Test
@DisplayName("Add operation test")
@RepeatedTest(5)
void addNumber(TestInfo testInfo) {
    Calculator calculator = new Calculator();
    Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
}
```

在上面的代码中，`addNumber()`测试将重复5次。请注意，重复测试的每次调用的行为就像一个常规`@Test`方法的执行，并且完全支持相同的生命周期回调和扩展。这意味着，`@BeforeEach`和`@AfterEach`注解的方法将被调用，他们适合在测试生命周期，为每个单独的调用。

```java
package com.github.tonydeng.junit5.examples;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @BeforeAll
    public static void init(){
        System.out.println("Before All init() method called");
    }

    @BeforeEach
    public void initEach(){
        System.out.println("Before Each initEach() method called");
    }

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(5)
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
        System.out.println("===addNumber testcase executed===");
    }

    @AfterEach
    public void cleanUpEach(){
        System.out.println("After Each cleanUpEach() method called");
    }

    @AfterAll
    public static void cleanUp(){
        System.out.println("After All cleanUp() method called");
    }
}
```

以上程序的输出：

```bash
Before All init() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

Before Each initEach() method called
===addNumber testcase executed===
After Each cleanUpEach() method called

After All cleanUp() method called
```

### 使用@RepeatedTest注解定制显示名称

除了指定重复次数之外，您还可以为每次重复提供自定义显示名称。此自定义显示名称可以是静态文本+动态占位符的组合。目前支持3名占有者：

- `{displayName}`：显示`@RepeatedTest`方法名称。
- `{currentRepetition}`：当前重复计数。
- `{totalRepetitions}`：重复的总数。

```java
@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample
{
    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(value = 5, name = "{displayName} - repetition {currentRepetition} of {totalRepetitions}")
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }
}
```

在IDE中运行该测试，将会产生类似下图的运行记录：


![RepeatedTest Run results](https://s1.ax1x.com/2017/10/10/8iA7q.png)

您可以使用两个预定义的格式之一，即`RepeatedTest.LONG_DISPLAY_NAME`和`RepeatedTest.SHORT_DISPLAY_NAME`。`SHORT_DISPLAY_NAME`是未指定的默认格式。

- `RepeatedTest.LONG_DISPLAY_NAME` - {displayName} :: repet {currentRepetition} {totalRepetitions}
- `RepeatedTest.SHORT_DISPLAY_NAME` - repetition {currentRepetition} {totalRepetitions}

```java
@Test
@DisplayName("Add operation test")
@RepeatedTest(value = 5, name = RepeatedTest.LONG_DISPLAY_NAME)
void addNumber(TestInfo testInfo) {
    Calculator calculator = new Calculator();
    Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
}
```

### 使用RepetitionInfo接口

[RepetitionInfo](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/RepetitionInfo.html)用于将关于当前重复的重复测试的信息注入到`@RepeatedTest`和`@BeforeEach`，和`@AfterEach`方法中。

```java
@RunWith(JUnitPlatform.class)
public class JUnit5AnnotationsExample {

    @BeforeEach
    public void initEach(RepetitionInfo info){
        int currentRepetition = info.getCurrentRepetition();
        int totalRepetitions = info.getTotalRepetitions();
        //Use information as needed
    }

    @Test
    @DisplayName("Add operation test")
    @RepeatedTest(value = 5, name="{displayName} :: repetition {currentRepetition} of {totalRepetitions}")
    void addNumber(TestInfo testInfo) {
        Calculator calculator = new Calculator();
        Assertions.assertEquals(2, calculator.add(1, 1), "1 + 1 should equal 2");
    }

    @AfterEach
    public void cleanUpEach(RepetitionInfo info){
        int currentRepetition = info.getCurrentRepetition();
        int totalRepetitions = info.getTotalRepetitions();
        //Use information as needed
    }
}
```

> 到目前为止，整个`@RepeatedTest`功能处于`Experimental`状态。将来可以更新甚至删除。

## @Disabled

[@Disabled](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Disabled.html)注解可用于禁用测试套件的测试方法。此注解可以应用于测试类以及单独的测试方法。

它只接受一个可选参数，指示该测试被禁用的原因。

### @Disabled测试类

当@Disabled通过测试类应用时，该类中的所有测试方法也将自动禁用。

```java
import org.junit.jupiter.api.Assumptions;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

@Disabled
public class AppTest {

    @Test
    void testOnDev()
    {
        System.setProperty("ENV", "DEV");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }

    @Test
    void testOnProd()
    {
        System.setProperty("ENV", "PROD");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }
}
```

> 注意运行次数：2/2（2跳过）。

### @Disabled测试方法

@Disabled用于表示注解的测试方法当前已禁用，不应执行。

```java
import org.junit.jupiter.api.Assumptions;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

public class AppTest {

    @Disabled("Do not run in lower environment")
    @Test
    void testOnDev()
    {
        System.setProperty("ENV", "DEV");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }

    @Test
    void testOnProd()
    {
        System.setProperty("ENV", "PROD");
        Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
    }
}
```

> 注意运行次数：2/2（1跳过）。

## @Tag

 [@Tag](http://junit.org/junit5/docs/current/api/org/junit/jupiter/api/Tag.html)可用于从测试计划中过滤测试用例。它可以帮助为不同环境，不同用例或任何特定要求创建多个不同的测试计划。您可以通过在测试计划OR中仅包括那些标记的测试来排除测试计划中的其他测试来执行一组测试。


### @Tag注解用法

您可以`@Tag`在测试类或测试方法或两者上应用注解。

```java
@Tag("development")
public class ClassATest
{
    @Test
    @Tag("userManagement")
    void testCaseA(TestInfo testInfo) {
    }
}
您也可以在单个测试用例上应用多个标签，以便将其包含在多个测试计划中。

public class ClassATest
{
    @Test
    @Tag("development")
    @Tag("production")
    void testCaseA(TestInfo testInfo) {
    }
}
```

### 使用@IncludeTags和@ExcludeTags创建测试计划

可以在测试计划中使用`@IncludeTags`或`@ExcludeTags`注解来过滤测试或包含测试。

```java
//@IncludeTags example

@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@IncludeTags("production")
public class JUnit5Example
{
}

//@ExcludeTags example

@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@ExcludeTags("production")
public class JUnit5Example
{
}
```

要添加多个标签，请将所需标注的字符串数组传递给所需的注解。

```java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@IncludeTags({"production","development"})
public class JUnit5Example
{
}
```

> 不能在单个测试计划中同时包含`@IncludeTags`和`@ExcludeTags`注解。

### @Tag示例

假设我们有3个测试，我们想在开发环境中运行所有3个测试; 但是想在生产中只运行一个。所以我们将标记测试如下：

```java
public class ClassATest
{
    @Test
    @Tag("development")
    @Tag("production")
    void testCaseA(TestInfo testInfo) { //run in all environments
    }
}

public class ClassBTest
{
    @Test
    @Tag("development")
    void testCaseB(TestInfo testInfo) {
    }
}

public class ClassCTest
{
    @Test
    @Tag("development")
    void testCaseC(TestInfo testInfo) {
    }
}
```

让我们为这两种环境创建测试计划。

#### 测试在生产环境中运行

```java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@IncludeTags("production")
public class ProductionTests
{
}
```

#### 测试在开发环境中运行

```java
@RunWith(JUnitPlatform.class)
@SelectPackages("com.github.tonydeng.junit5.examples")
@IncludeTags("development")
public class DevelopmentTests
{
}
```
