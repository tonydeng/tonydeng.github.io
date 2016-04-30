layout: post
title: Java Stream详解
published: true
comments: true
sharing: true
footer: true
categories: [程序设计]
tags: [java, stream]
date: 2016-04-25 16:59:03
keywords: java stream
---

![java8 stream api](/images/stream-api-java8.png)

Stream是 Java 8新增加的类，用来补充集合类。

Stream代表数据流，流中的数据元素的数量可能是有限的，也可能是无限的。

Stream和其它集合类的区别在于：其它集合类主要关注与有限数量的数据的访问和有效管理(增删改)，而Stream并没有提供访问和管理元素的方式，而是通过声明数据源的方式，利用可计算的操作在数据源上执行，当然`BaseStream.iterator()` 和 `BaseStream.spliterator()`操作提供了遍历元素的方法。

Java Stream提供了提供了串行和并行两种类型的流，保持一致的接口，提供函数式编程方式，以管道方式提供中间操作和最终执行操作，为Java语言的集合提供了现代语言提供的类似的高阶函数操作，简化和提高了Java集合的功能。

本文首先介绍Java Stream的特点，然后按照功能分类逐个介绍流的中间操作和终点操作，最后会介绍第三方为Java Stream做的扩展。

<!-- more -->

## 介绍


本节翻译整理自[Java8 Stream Javadoc](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html
)，并对流的这些特性做了进一步的解释。

Stream接口还包含几个基本类型的子接口如`IntStream`, `LongStream` 和 `DoubleStream`。

关于流和其它集合具体的区别，可以参照下面的列表：

1. 不存储数据。流是基于数据源的对象，它本身不存储数据元素，而是通过管道将数据源的元素传递给操作。
2. 函数式编程。流的操作不会修改数据源，例如filter不会将数据源中的数据删除。
3. 延迟操作。流的很多操作如filter,map等中间操作是延迟执行的，只有到终点操作才会将操作顺序执行。
4. 可以解绑。对于无限数量的流，有些操作是可以在有限的时间完成的，比如limit(n) 或 findFirst()，这些操作可是实现"短路"(Short-circuiting)，访问到有限的元素后就可以返回。
5. 纯消费。流的元素只能访问一次，类似Iterator，操作没有回头路，如果你想从头重新访问流的元素，对不起，你得重新生成一个新的流。

流的操作是以管道的方式串起来的。流管道包含一个数据源，接着包含零到N个中间操作，最后以一个终点操作结束。

![java stream](/images/streams.png)

### 并行 Parallelism

所有的流操作都可以串行执行或者并行执行。

除非显示地创建并行流，否则Java库中创建的都是串行流。 `Collection.stream()`为集合创建串行流而`Collection.parallelStream()`为集合创建并行流。`IntStream.range(int, int)`创建的是串行流。通过`parallel()`方法可以将串行流转换成并行流,sequential()方法将流转换成串行流。

除非方法的Javadoc中指明了方法在并行执行的时候结果是不确定(比如`findAny`、`forEach`)，否则串行和并行执行的结果应该是一样的。

### Non-interference

流可以从非线程安全的集合中创建，当流的管道执行的时候，非concurrent数据源不应该被改变。下面的代码会抛出`java.util.ConcurrentModificationException`异常：

```java
List<String> l = new ArrayList(Arrays.asList("one", "two"));
Stream<String> sl = l.stream();
sl.forEach(s -> l.add("three"));
```

在设置中间操作的时候，可以更改数据源，只有在执行终点操作的时候，才有可能出现并发问题(抛出异常，或者不期望的结果)，比如下面的代码不会抛出异常：

```java
List<String> l = new ArrayList(Arrays.asList("one", "two"));
Stream<String> sl = l.stream();
l.add("three");
sl.forEach(System.out::println);
```

对于concurrent数据源，不会有这样的问题，比如下面的代码很正常：

```java
List<String> l = new CopyOnWriteArrayList<>(Arrays.asList("one", "two"));
Stream<String> sl = l.stream();
sl.forEach(s -> l.add("three"));
```

虽然我们上面例子是在终点操作中对非并发数据源进行修改，但是非并发数据源也可能在其它线程中修改，同样会有并发问题。

### 无状态 Stateless behaviors

大部分流的操作的参数都是函数式接口，可以使用Lambda表达式实现。它们用来描述用户的行为，称之为行为参数(behavioral parameters)。

如果这些行为参数有状态，则流的操作的结果可能是不确定的，比如下面的代码:

```java
List<String> l = new ArrayList(Arrays.asList("one", "two", ……));
class State {
    boolean s;
}
final State state = new State();
Stream<String> sl = l.stream().map(e -> {
    if (state.s)
        return "OK";
    else {
        state.s = true;
        return e;
    } 
});
sl.forEach(System.out::println);
```

上面的代码在并行执行时多次的执行结果可能是不同的。这是因为这个lambda表达式是有状态的。

### 排序 Ordering

某些流的返回的元素是有确定顺序的，我们称之为 *encounter order*。这个顺序是流提供它的元素的顺序，比如数组的encounter order是它的元素的排序顺序，List是它的迭代顺序(iteration order)，对于HashSet,它本身就没有encounter order。

一个流是否是encounter order主要依赖数据源和它的中间操作，比如数据源List和Array上创建的流是有序的(ordered)，但是在HashSet创建的流不是有序的。

`sorted()`方法可以将流转换成有序的，`unordered`可以将流转换成无序的。
除此之外，一个操作可能会影响流的有序,比如`map`方法，它会用不同的值甚至类型替换流中的元素，所以输入元素的有序性已经变得没有意义了，但是对于`filter`方法来说，它只是丢弃掉一些值而已，输入元素的有序性还是保障的。

对于串行流，流有序与否不会影响其性能，只是会影响确定性(determinism)，无序流在多次执行的时候结果可能是不一样的。

对于并行流，去掉有序这个约束可能会提供性能，比如`distinct`、`groupingBy`这些聚合操作。

### 结合性 Associativity

一个操作或者函数op满足结合性意味着它满足下面的条件：

```
(a op b) op c == a op (b op c)
```

对于并发流来说，如果操作满足结合性，我们就可以并行计算：

```java
a op b op c op d == (a op b) op (c op d)
```

比如`min`、`max`以及字符串连接都是满足结合性的。

## 创建Stream

可以通过多种方式创建流：

1. 通过集合的`stream()`方法或者`parallelStream()`，比如`Arrays.asList(1,2,3).stream()`。
2. 通过`Arrays.stream(Object[])`方法, 比如`Arrays.stream(new int[]{1,2,3})`。
3. 使用流的静态方法，比如`Stream.of(Object[])`, `IntStream.range(int, int)` 或者 `Stream.iterate(Object, UnaryOperator)，如Stream.iterate(0, n -> n * 2)`，或者`generate(Supplier<T> s)`如`Stream.generate(Math::random)`。
4. `BufferedReader.lines()`从文件中获得行的流。
5. `Files`类的操作路径的方法，如`list`、`find`、`walk`等。
6. 随机数流`Random.ints()`。
7. 其它一些类提供了创建流的方法，如`BitSet.stream()`, `Pattern.splitAsStream(java.lang.CharSequence)`, 和 `JarFile.stream()`。
8. 更底层的使用`StreamSupport`，它提供了将`Spliterator`转换成流的方法。

## 中间操作 intermediate operations

中间操作会返回一个新的流，并且操作是延迟执行的(lazy)，它不会修改原始的数据源，而且是由在终点操作开始的时候才真正开始执行。
这个Scala集合的转换操作不同，Scala集合转换操作会生成一个新的中间集合，显而易见Java的这种设计会减少中间对象的生成。

下面介绍流的这些中间操作：

### distinct

`distinct`保证输出的流中包含唯一的元素，它是通过`Object.equals(Object)`来检查是否包含相同的元素。

```java
List<String> l = Stream.of("a","b","c","b")
        .distinct()
        .collect(Collectors.toList());
System.out.println(l); //[a, b, c]
```

### filter

`filter`返回的流中只包含满足断言(predicate)的数据。

下面的代码返回流中的偶数集合。

```java
List<Integer> l = IntStream.range(1,10)
        .filter( i -> i % 2 == 0)
        .boxed()
        .collect(Collectors.toList());
System.out.println(l); //[2, 4, 6, 8]
```

### map

`map`方法将流中的元素映射成另外的值，新的值类型可以和原来的元素的类型不同。

下面的代码中将字符元素映射成它的哈希码(ASCII值)。

```java
List<Integer> l = Stream.of('a','b','c')
        .map( c -> c.hashCode())
        .collect(Collectors.toList());
System.out.println(l); //[97, 98, 99]
```

### flatmap

`flatmap`方法混合了`map` + `flattern`的功能，它将映射后的流的元素全部放入到一个新的流中。它的方法定义如下：

```java
<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)
```

可以看到`mapper`函数会将每一个元素转换成一个流对象，而flatMap方法返回的流包含的元素为`mapper`生成的所有流中的元素。

下面这个例子中将一首唐诗生成一个按行分割的流，然后在这个流上调用`flatmap`得到单词的小写形式的集合，去掉重复的单词然后打印出来。

```java
String poetry = "Where, before me, are the ages that have gone?\n" +
        "And where, behind me, are the coming generations?\n" +
        "I think of heaven and earth, without limit, without end,\n" +
        "And I am all alone and my tears fall down.";
Stream<String> lines = Arrays.stream(poetry.split("\n"));
Stream<String> words = lines.flatMap(line -> Arrays.stream(line.split(" ")));
List<String> l = words.map( w -> {
    if (w.endsWith(",") || w.endsWith(".") || w.endsWith("?"))
        return w.substring(0,w.length() -1).trim().toLowerCase();
    else
        return w.trim().toLowerCase();
}).distinct().sorted().collect(Collectors.toList());
System.out.println(l); //[ages, all, alone, am, and, are, before, behind, coming, down, earth, end, fall, generations, gone, have, heaven, i, limit, me, my, of, tears, that, the, think, where, without]
```
`flatMapToDouble`、`flatMapToInt`、`flatMapToLong`提供了转换成特定流的方法。

### limit

`limit`方法指定数量的元素的流。对于串行流，这个方法是有效的，这是因为它只需返回前n个元素即可，但是对于有序的并行流，它可能花费相对较长的时间，如果你不在意有序，可以将有序并行流转换为无序的，可以提高性能。

```
List<Integer> l = IntStream.range(1,100).limit(5)
        .boxed()
        .collect(Collectors.toList());
System.out.println(l);//[1, 2, 3, 4, 5]
```

### peek

`peek`方法方法会使用一个`Consumer`消费流中的元素，但是返回的流还是包含原来的流中的元素。

```java
String[] arr = new String[]{"a","b","c","d"};
Arrays.stream(arr)
        .peek(System.out::println) //a,b,c,d
        .count();
```

### sorted

`sorted()`将流中的元素按照自然排序方式进行排序，如果元素没有实现Comparable，则终点操作执行时会抛出`java.lang.ClassCastException`异常。
`sorted(Comparator<? super T> comparator)`可以指定排序的方式。

对于有序流，排序是稳定的。对于非有序流，不保证排序稳定。

```java
String[] arr = new String[]{"b_123","c+342","b#632","d_123"};
List<String> l  = Arrays.stream(arr)
        .sorted((s1,s2) -> {
            if (s1.charAt(0) == s2.charAt(0))
                return s1.substring(2).compareTo(s2.substring(2));
            else
                return s1.charAt(0) - s2.charAt(0);
        })
        .collect(Collectors.toList());
System.out.println(l); //[b_123, b#632, c+342, d_123]
```

### skip

`skip`返回丢弃了前n个元素的流，如果流中的元素小于或者等于n，则返回空的流。

## 终点操作 terminal operations

### Match

```java
public boolean 	allMatch(Predicate<? super T> predicate)
public boolean 	anyMatch(Predicate<? super T> predicate)
public boolean 	noneMatch(Predicate<? super T> predicate)
```

这一组方法用来检查流中的元素是否满足断言。

`allMatch`只有在所有的元素都满足断言时才返回true,否则flase,流为空时总是返回true

`anyMatch`只有在任意一个元素满足断言时就返回true,否则flase,

`noneMatch`只有在所有的元素都不满足断言时才返回true,否则flase,

```
System.out.println(Stream.of(1,2,3,4,5).allMatch( i -> i > 0)); //true
System.out.println(Stream.of(1,2,3,4,5).anyMatch( i -> i > 0)); //true
System.out.println(Stream.of(1,2,3,4,5).noneMatch( i -> i > 0)); //false
System.out.println(Stream.<Integer>empty().allMatch( i -> i > 0)); //true
System.out.println(Stream.<Integer>empty().anyMatch( i -> i > 0)); //false
System.out.println(Stream.<Integer>empty().noneMatch( i -> i > 0)); //true
```

### count

`count`方法返回流中的元素的数量。它实现为：

```java
mapToLong(e -> 1L).sum();
```


### collect

```java
<R,A> R 	collect(Collector<? super T,A,R> collector)
<R> R 	collect(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner)
```

使用一个`collector`执行`mutable reduction`操作。辅助类`Collectors`提供了很多的`collector`，可以满足我们日常的需求，你也可以创建新的`collector`实现特定的需求。它是一个值得关注的类，你需要熟悉这些特定的收集器，如聚合类`averagingInt`、最大最小值`maxBy` `minBy`、计数`counting`、分组`groupingBy`、字符串连接`joining`、分区`partitioningBy`、汇总`summarizingInt`、化简`reducing`、转换`toXXX`等。

第二个提供了更底层的功能，它的逻辑类似下面的伪代码：

```java
R result = supplier.get();
for (T element : this stream)
    accumulator.accept(result, element);
return result;
```

例子：

```java
List<String> asList = stringStream.collect(ArrayList::new, ArrayList::add,
                                           ArrayList::addAll);
String concat = stringStream.collect(StringBuilder::new, StringBuilder::append,
                                     StringBuilder::append)
                            .toString();
```

### find

`findAny()`返回任意一个元素，如果流为空，返回空的`Optional`，对于并行流来说，它只需要返回任意一个元素即可，所以性能可能要好于`findFirst()`，但是有可能多次执行的时候返回的结果不一样。
`findFirst()`返回第一个元素，如果流为空，返回空的`Optional`。

### forEach、forEachOrdered

`forEach`遍历流的每一个元素，执行指定的action。它是一个终点操作，和peek方法不同。这个方法不担保按照流的`encounter order`顺序执行，如果对于有序流按照它的`encounter order`顺序执行，你可以使用`forEachOrdered`方法。

```java
Stream.of(1,2,3,4,5).forEach(System.out::println);
```

### max、min

`max`返回流中的最大值，

`min`返回流中的最小值。

### reduce

`reduce`是常用的一个方法，事实上很多操作都是基于它实现的。

它有几个重载方法：

```java
pubic Optional<T> 	reduce(BinaryOperator<T> accumulator)
pubic T 	reduce(T identity, BinaryOperator<T> accumulator)
pubic <U> U 	reduce(U identity, BiFunction<U,? super T,U> accumulator, BinaryOperator<U> combiner)
```

第一个方法使用流中的第一个值作为初始值，后面两个方法则使用一个提供的初始值。

```java
Optional<Integer> total = Stream.of(1,2,3,4,5).reduce( (x, y) -> x +y);
Integer total2 = Stream.of(1,2,3,4,5).reduce(0, (x, y) -> x +y);
```

值得注意的是`accumulator`应该满足结合性(associative)。

### toArray()

将流中的元素放入到一个数组中。

## 组合

`concat`用来连接类型一样的两个流。

```java
public static <T> Stream<T> 	concat(Stream<? extends T> a, Stream<? extends T> b)
```

## 转换

`toArray`方法将一个流转换成数组，而如果想转换成其它集合类型，西需要调用`collect`方法，利用`Collectors.toXXX`方法进行转换：

```java
public static <T,C extends Collection<T>> Collector<T,?,C> 	toCollection(Supplier<C> collectionFactory)
public static …… 	toConcurrentMap(……)
public static <T> Collector<T,?,List<T>> 	toList()
public static …… 	toMap(……)
public static <T> Collector<T,?,Set<T>> 	toSet()
```

## 更进一步

虽然Stream提供了很多的操作，但是相对于Scala等语言，似乎还少了一些。一些开源项目提供了额外的一些操作，比如[protonpack](https://github.com/poetix/protonpack)项目提供了下列方法：

* takeWhile and takeUntil
* skipWhile and skipUntil
* zip and zipWithIndex
* unfold
* MapStream
* aggregate
* Streamable
* unique collector

[java8-utils](https://github.com/NitorCreations/java8-utils) 也提供了一些有益的辅助方法。

## 参考文档

https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html
http://www.leveluplunch.com/java/examples/
https://github.com/poetix/protonpack
https://github.com/NitorCreations/java8-utils