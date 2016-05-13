layout: post
title: 使用Junit的一些的高级用法
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [juit, rule, java, testcase]
date: 2016-05-11 18:39:23
keywords: Junit Rule
---

![Junit](/images/blog/junit.png)

`Junit` 是Java开发领域中非常普遍的单元测试框架，不过，大部分的使用者仅仅只是使用它一部分的功能。

先整理一部分相对不常用，但是个人觉得非常有用的功能（其他的慢慢补充），希望对大家有所帮助。

<!-- more -->

## Rule注解

比如，我们可以使用 @Role 注解来提高我们写单元测试的效率。

`@Rule` 这个注解是 `Junit4` 的新特性，我们可以去看看Junit的关于[Rule的使用例子](https://github.com/junit-team/junit4/wiki/Rules)。

利用 `@Rule` 我们可以扩展 `Junit` 的功能，在执行case的时候加入测试者特有的操作，而不影响原有case的代码，减少了特有操作和test case原逻辑的耦合。

`@Rule` 只能注解在字段中，该字段必须是 `public` 的并且类型必须实现了 `TestRule` 接口或者 `MethodRule` 接口。

Junit 4.9之后还加入了一个 `@ClassRule` 注解。相对 `@Rule` 来说， `@ClassRule` 是一个类级别的注解。就像 `@Before` 与 `@BeforeClass` 的区别。

### 通过@Rule注解生成临时文件或临时文件夹

有时候程序运行时必须生成文件或文件夹，往往需要写不少代码来实现这个功能。我们可以使用 TemporaryFolder 来在测试的时候创建文件和目录，最爽的是它在测试运行结束之后会将测试时创建的文件和目录的自动删除。

```java
import org.apache.commons.io.FileUtils;
import org.junit.Assert;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.TemporaryFolder;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.IOException;

/**
 * Created by tonydeng on 16/5/10.
 */
public class JunitRuleTest {
    private static final Logger log = LoggerFactory.getLogger(JunitRuleTest.class);

    @Rule
    public TemporaryFolder tempFolder = new TemporaryFolder();

    @Test
    public void testFileCreateAndWrite() throws IOException {
        File file = tempFolder.newFile("simple.txt");
        log.info("temp file:'{}'", file.getPath());
        FileUtils.writeStringToFile(file, "Junit Rules!");
        String line = FileUtils.readFileToString(file);
        Assert.assertThat(line, is("Junit Rules!"));
    }
}
```

测试完成之后，生成的这个simple.txt文件就会自动删除了。

### 取得当前的测试方法名称

想要取得执行中的测试方法名的时候，通过@Rule注解TestName类的实例化对象可以取得。

```java
 @Rule

 public TestName name = new TestName();

 @Test 

 public void testMethodName() {

    log.info("Test method name: {}",name.getMethodName());

 }
```

测试结果：

```
Test method name:'testMethodName'
```

### 在一个运行测试方法过程中收集多个错误信息

使用 `ErrorCollector` 类，可以在一个测试方法中收集多个测试错误。也就是说，一个测试方法执行中，不会在第一个确认出错后就停止执行。使用 `ErrorCollector` 可以在所有点确认完后统一报出。

比如，有三个点需要check，但是又不想出错了就马上退出测试，可以尝试下面的方式。

```java
@Rule
public ErrorCollector collector = new ErrorCollector();

@Test
public void testMoreCollector() {
	String s = null;
	collector.checkThat("Value should not be null", null, is(s));

	s = "";
	collector.checkThat("Value should have the length of 1",s.length(),is(1));

	s = "Junit!";
	collector.checkThat("Value should have the length of 10",s.length(),is(10));
}
```


上面的测试会报这样的错误信息：

```java
Failed tests:   testMoreCollector(com.github.tonydeng.demo.java8.JunitRuleTest): Value should have the length of 1(..)
  testMoreCollector(com.github.tonydeng.demo.java8.JunitRuleTest): Value should have the length of 10(..)
```

或者自己手工捕获异常，添加到 `ErrorCollector` 中，通过 `addError` 添加的错误信息：。

```java
@Test
public void testErrorCollector(){
	collector.addError(new Throwable("first thing went wrong"));
	collector.addError(new Throwable("second thing went wrong"));
}
```

错误信息：

```java
Tests in error:
  testErrorCollector(com.github.tonydeng.demo.java8.JunitRuleTest): first thing went wrong
  testErrorCollector(com.github.tonydeng.demo.java8.JunitRuleTest): second thing went wrong
```

### 设置执行最长时间

我们有时候要对某些方法调用的时长有要求，如果超出某些时长就算是不符合要求。那我们平时怎么来测试呢？

> 计算一下调用方法之前和之后的时间差，如果超出某个值，就算不符合要求。

的确，使用上述的方式也可以达到目的，但是还是有太多多余的代码要写了。完全可以尝试一下 `Junit` 的 `Timeout`，非常简单就达到你的目的了。

比如，我们先设置一个5秒的超时，然后用一个无限循环的方法来测试。

```java
 	@Rule
    public Timeout timeout = Timeout.seconds(5);

    @Test
    public void testTimeout() {
        while (true) ;
    }
```

执行结果：

```java
org.junit.runners.model.TestTimedOutException: test timed out after 5 seconds
```

### 使用RuleChain


RuleChain提供一种将多个TestRule串在一起执行的机制。这在JUnit 4.10以后的版本中可以使用。需要根据特定顺序执行多个处理的时候，用RuleChain可以提高效率。

```java
public static class UseRuleChain {
    @Rule
    public TestRule chain = RuleChain
                           .outerRule(new LoggingRule("outer rule"))
                           .around(new LoggingRule("middle rule"))
                           .around(new LoggingRule("inner rule"));

    @Test
    public void example() {
        assertTrue(true);
    }
}
```

执行结果

```java
starting outer rule
starting middle rule
starting inner rule
finished inner rule
finished middle rule
finished outer rule
```

## Parameterized注解

如果我们需要对我们的测试方法进行参数化，也就是只写一个测试方法，把若干种情况作为参数传递进去，一次性完成测试。那我们该怎么办？

可以使用 `Parameterized` 相关的注解来解决这个问题。

来一个简单的计算斐波纳契数的例子：


```java
/**
 * Created by tonydeng on 16/5/12.
 */
@RunWith(Parameterized.class)
public class FibonacciNumbersTest {
    private static final Logger log = LoggerFactory.getLogger(FibonacciNumbersTest.class);

    public static Collection<Integer[]> data() {
        return Arrays.asList(new Integer[][]{
                {0, 0}, {1, 1}, {2, 1},
                {3, 2}, {4, 3}, {5, 5},
                {6, 8}});
    }

    private int value;
    private int expected;

    public FibonacciNumbersTest(int input, int expected) {
        value = input;
        this.expected = expected;
    }
    
    @Test
    public void fibonacciNumberCall() {
        log.info("expected {} fib(value) {}", expected, fib(value));
        Assert.assertEquals(expected, fib(value));
    }
    
    public static int fib(int n) {
        if (n < 2) {
            return n;
        } else {
            return fib(n - 1) + fib(n - 2);
        }
    }
}
```

或者，这么来写：


```java
/**
 * Created by tonydeng on 16/5/12.
 */
@RunWith(Parameterized.class)
public class FibonacciNumbersTest {
    private static final Logger log = LoggerFactory.getLogger(FibonacciNumbersTest.class);

    @Parameterized.Parameters(name = "{index}: fib({0}={1})")
    public static Collection<Integer[]> data() {
        return Arrays.asList(new Integer[][]{
                {0, 0}, {1, 1}, {2, 1},
                {3, 2}, {4, 3}, {5, 5},
                {6, 8}});
    }

    @Parameterized.Parameter
    public int fInput;

    @Parameterized.Parameter(value = 1)
    public int fExpected;

    @Test
    public void testParemeter(){
        log.info("fExpected {} fib(fInput) {}", fExpected, fib(fInput));
        Assert.assertEquals(fExpected, fib(fInput));
    }

    public static int fib(int n) {
        if (n < 2) {
            return n;
        } else {
            return fib(n - 1) + fib(n - 2);
        }
    }
}
```

上面的两个例子中，都是利用了 `data()` 方法构建了各个测试方法的参数，其中返回值的第一个是input参数，第二个是expected参数。

## 参考

[junit4 wiki via github](https://github.com/junit-team/junit4/wiki)

[JUnit Tutorial](http://www.codeaffine.com/2014/09/24/junit-nutshell-junit-tutorial/)