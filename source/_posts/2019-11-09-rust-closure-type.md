layout: post
title: Rust的闭包类型(Fn, FnMut, FnOne的区别)
published: true
comments: true
sharing: true
footer: true
categories: [Rust入坑之旅]
tags: [Rust入坑之旅, 闭包, Rust, Closure, Fn, FnOne, FnMut]
date: 2019-11-09 00:27:10
keywords: Rust入坑之旅, 闭包, Rust, Closure, Fn, FnOne, FnMut
---

![Rust Logo](/images/blog/rust/rust-language.jpg)

对于支持函数式编程的语言来说，支持闭包是一个非常必要的能力。但是我们在`Rust`中使用闭包的时候经常会出现一些莫名其妙、无法理解的错误。

你可能会这样的经历：你非常兴奋的使用闭包这个特性写了一段自认牛逼的代码，发现根本无法编译通，修改了好几次，终于在编译器的友（残）好（暴）提（敲）示（打）下，终于编译完成，执行成功了，但是下一次依然碰到同样的遭遇。

为了避免持续出现这样的情况，我们来系统的看一看`Rust`中的闭包。

<!-- more -->

## 闭包是什么？

先来看看[维基百科](https://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))上的描述：

> 在计算机科学中，闭包（英语：`Closure`），又称词法闭包（`Lexical Closure`）或函数闭包（`function closures`），是**引用了自由变量的函数**。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

闭包的概念出现于60年代，最早实现闭包的程序语言是`Scheme`。之后，闭包被广泛使用于函数式编程语言如`ML`语言和`LISP`。很多命令式程序语言也开始支持闭包。

可以看到，第一句就已经说明了什么是闭包：闭包是引用了自由变量的函数。所以，闭包是一种特殊的函数。

在`Rust`里，闭包被分为了三种类型，列举如下

- [Fn(&self)](https://doc.rust-lang.org/std/ops/trait.Fn.html)
- [FnMut(&mut self)](https://doc.rust-lang.org/std/ops/trait.FnMut.html)
- [FnOnce(self)](https://doc.rust-lang.org/std/ops/trait.FnOnce.html)

在`rust`中，函数和闭包都是实现了`Fn`、`FnMut`或`FnOnce`特质（`trait`）的类型。任何实现了这三种特质其中一种的类型的对象，都是 可调用对象 ，都能像函数和闭包一样通过这样`name()`的形式调用，`()`在`rust`中是一个操作符，操作符在`rust`中是可以重载的。`rust`的操作符重载是通过实现相应的`trait`来实现，而`()`操作符的相应`trait`就是`Fn`、`FnMut`和`FnOnce`，所以，任何实现了这三个`trait`中的一种的类型，其实就是重载了`()`操作符。

## `Rust`中三种闭包的定义：

![Rust的三种闭包类型](/images/blog/rust/rust-closure.jpg)

> 图片来源于[知乎-Rust编程专栏](https://zhuanlan.zhihu.com/p/64417628)

### `FnOnce`

#### 标准库定义

```rust
#[lang = "fn_once"]
pub trait FnOnce<Args> {
    type Output;
    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
```

参数类型是`self`，所以，这种类型的闭包会获取变量的所有权，生命周期只能是当前作用域，之后就会被释放了。

#### `FnOnce`例子：

```rust
#[derive(Debug)]
struct E {
    a: String,
}

impl Drop for E {
    fn drop(&mut self) {
        println!("destroyed struct E");
    }
}

fn fn_once<F>(func: F) where F: FnOnce() {
    println!("fn_once begins");
    func();
    println!("fn_once ended");
}

fn main() {
    let e = E { a: "fn_once".to_string() };
    // 这样加个move，看看程序执行输出顺序有什么不同
    // let f = move || println!("fn once calls: {:?}", e);
    let f = || println!("fn once closure calls: {:?}", e);
    fn_once(f);
    println!("main ended");
}
```

打印结果如下：

```rust
fn_once begins
fn once closure calls: E { a: "fn_once" }
fn_once ended
main ended
destroyed struct E
```

> [FnOnce类型闭包-Rust playground例子](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=%23%5Bderive(Debug)%5D%0Astruct%20E%20%7B%0A%20%20%20%20a%3A%20String%2C%0A%7D%0A%0Aimpl%20Drop%20for%20E%20%7B%0A%20%20%20%20fn%20drop(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20println!(%22destroyed%20struct%20E%22)%3B%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20fn_once%3CF%3E(func%3A%20F)%20where%20F%3A%20FnOnce()%20%7B%0A%20%20%20%20println!(%22fn_once%20begins%22)%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20println!(%22fn_once%20ended%22)%3B%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20e%20%3D%20E%20%7B%20a%3A%20%22fn_once%22.to_string()%20%7D%3B%0A%20%20%20%20%2F%2F%20%E8%BF%99%E6%A0%B7%E5%8A%A0%E4%B8%AAmove%EF%BC%8C%E7%9C%8B%E7%9C%8B%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C%E8%BE%93%E5%87%BA%E9%A1%BA%E5%BA%8F%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C%0A%20%20%20%20%2F%2F%20let%20f%20%3D%20move%20%7C%7C%20println!(%22fn%20once%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20let%20f%20%3D%20%7C%7C%20println!(%22fn%20once%20closure%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20fn_once(f)%3B%0A%20%20%20%20println!(%22main%20ended%22)%3B%0A%7D)

但是如果闭包运行两次，比如:

```rust
fn fn_once<F>(func: F) where F: FnOnce() {
    println!("fn_once begins");
    func();
    func();
    println!("fn_once ended");
}
```

则编译器就报错了，类似这样：

```rust
error[E0382]: use of moved value: `func`
  --> src/main.rs:15:5
   |
12 | fn fn_once<F>(func: F) where F: FnOnce() {
   |            -  ---- move occurs because `func` has type `F`, which does not implement the `Copy` trait
   |            |
   |            consider adding a `Copy` constraint to this type argument
13 |     println!("fn_once begins");
14 |     func();
   |     ---- value moved here
15 |     func();
   |     ^^^^ value used here after move

error: aborting due to previous error
```

> [FnOnce类型闭包错误-Rust playgroound例子](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=%23%5Bderive(Debug)%5D%0Astruct%20E%20%7B%0A%20%20%20%20a%3A%20String%2C%0A%7D%0A%0Aimpl%20Drop%20for%20E%20%7B%0A%20%20%20%20fn%20drop(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20println!(%22destroyed%20struct%20E%22)%3B%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20fn_once%3CF%3E(func%3A%20F)%20where%20F%3A%20FnOnce()%20%7B%0A%20%20%20%20println!(%22fn_once%20begins%22)%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20println!(%22fn_once%20ended%22)%3B%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20e%20%3D%20E%20%7B%20a%3A%20%22fn_once%22.to_string()%20%7D%3B%0A%20%20%20%20%2F%2F%20%E8%BF%99%E6%A0%B7%E5%8A%A0%E4%B8%AAmove%EF%BC%8C%E7%9C%8B%E7%9C%8B%E7%A8%8B%E5%BA%8F%E6%89%A7%E8%A1%8C%E8%BE%93%E5%87%BA%E9%A1%BA%E5%BA%8F%E6%9C%89%E4%BB%80%E4%B9%88%E4%B8%8D%E5%90%8C%0A%20%20%20%20%2F%2F%20let%20f%20%3D%20move%20%7C%7C%20println!(%22fn%20once%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20let%20f%20%3D%20%7C%7C%20println!(%22fn%20once%20closure%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20fn_once(f)%3B%0A%20%20%20%20println!(%22main%20ended%22)%3B%0A%7D)

这是为什么呢？

还是回到`FnOnce`的定义，参数类型是`self`，所以在`func`第一次执行完之后，之前捕获的变量已经被释放了，所以已经无法在执行第二次了。所以，如果要运行多次，可以使用`FnMut\Fn`。

### `FnMut`

#### 标准库定义

```rust
#[lang = "fn_mut"]
pub trait FnMut<Args>: FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}
```

参数类型是`&mut self`，所以，这种类型的闭包是可变借用，会改变变量，但不会释放该变量。所以可以运行多次。

#### `FnMut`例子

和上面的例子差不多，有两个地方要改下:

```rust
fn fn_mut<F>(mut func: F) where F: FnMut() {
    func();
    func();
}

// ...
let mut e = E { a: "fn_once".to_string() };
let f = || { println!("FnMut closure calls: {:?}", e); e.a = "fn_mut".to_string(); };
// ...
```

打印结果如下：

```rust
fn_mut begins
fn mut closure calls: E { a: "fn_mut" }
fn mut closure calls: E { a: "fn_mut" }
fn_mut ended
main ended
destroyed struct E
```

> [FnMut类型闭包-Rust playground例子](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=%23%5Bderive(Debug)%5D%0Astruct%20E%20%7B%0A%20%20%20%20a%3A%20String%2C%0A%7D%0A%0Aimpl%20Drop%20for%20E%20%7B%0A%20%20%20%20fn%20drop(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20println!(%22destroyed%20struct%20E%22)%3B%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20fn_mut%3CF%3E(mut%20func%3A%20F)%20where%20F%3A%20FnMut()%20%7B%0A%20%20%20%20println!(%22fn_mut%20begins%22)%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20println!(%22fn_mut%20ended%22)%3B%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20e%20%3D%20E%20%7B%20a%3A%20%22fn_mut%22.to_string()%20%7D%3B%0A%20%20%20%20let%20f%20%3D%20%7C%7C%20println!(%22fn%20mut%20closure%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20fn_mut(f)%3B%0A%20%20%20%20println!(%22main%20ended%22)%3B%0A%7D)

可以看出`FnMut`类型的闭包是可以运行多次的，且可以修改捕获变量的值。

### Fn

#### 标准库定义

```rust
#[lang = "fn"]
pub trait Fn<Args>: FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}
```

参数类型是`&self`，所以，这种类型的闭包是不可变借用，不会改变变量，也不会释放该变量。所以可以运行多次。

#### `Fn`例子

```rust
fn fn_immut<F>(func: F) where F: Fn() {
    func();
    func();
}

// ...
let e = E { a: "fn".to_string() };
let f = || { println!("Fn closure calls: {:?}", e); };
fn_immut(f);
// ...
```

打印结果如下：

```rust
fn_imut begins
fn closure calls: E { a: "fn" }
fn closure calls: E { a: "fn" }
fn_imut ended
main ended
destroyed struct E
```

可以看出`Fn`类型的闭包是可以运行多次的，但不可以修改捕获变量的值。

> [Fn类型闭包-Rust playground例子](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&code=%23%5Bderive(Debug)%5D%0Astruct%20E%20%7B%0A%20%20%20%20a%3A%20String%2C%0A%7D%0A%0Aimpl%20Drop%20for%20E%20%7B%0A%20%20%20%20fn%20drop(%26mut%20self)%20%7B%0A%20%20%20%20%20%20%20%20println!(%22destroyed%20struct%20E%22)%3B%0A%20%20%20%20%7D%0A%7D%0A%0Afn%20fn_imut%3CF%3E(func%3A%20F)%20where%20F%3A%20Fn()%20%7B%0A%20%20%20%20println!(%22fn_imut%20begins%22)%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20func()%3B%0A%20%20%20%20println!(%22fn_imut%20ended%22)%3B%0A%7D%0A%0Afn%20main()%20%7B%0A%20%20%20%20let%20e%20%3D%20E%20%7B%20a%3A%20%22fn%22.to_string()%20%7D%3B%0A%20%20%20%20let%20f%20%3D%20%7C%7C%20println!(%22Fn%20closure%20calls%3A%20%7B%3A%3F%7D%22%2C%20e)%3B%0A%20%20%20%20fn_imut(f)%3B%0A%20%20%20%20println!(%22main%20ended%22)%3B%0A%7D)

## 常见的错误

有时候在使用`Fn/FnMut`这里类型的闭包，编译器经常会给出这样的错误:

```rust
# ...
cannot move out of captured variable in an Fn(FnMut) closure
# ...
```

看下如何复现这种情形:

```rust
fn main() {
    fn fn_immut<F>(f: F) where F: Fn() -> String {
        println!("calling Fn closure from fn, {}", f());
    }

    let a = "Fn".to_string();
    fn_immut(|| a); // 闭包返回一个字符串
}
```

这样写就会出现上面的那种错误。但如何修复呢？

```rust
fn_immut(|| a.clone());
```

但原因是什么呢？

只要稍稍改下上面的代码，运行一次，编译器给出的错误就很明显了:

```rust
fn main() {
    fn fn_immut<F>(f: F) where F: Fn() -> String {
        println!("calling Fn closure from fn, {}", f());
    }

    let a = "Fn".to_string();
    let f = || a;
    fn_immut(f);
}
```

编译器给出的错误如下:

```rust
7 |     let f = move || a;
  |             ^^^^^^^^-
  |             |       |
  |             |       closure is `FnOnce` because it moves the variable `a` out of its environment
  |             this closure implements `FnOnce`, not `Fn`
8 |     fn_immut(f);
  |     -------- the requirement to implement `Fn` derives from here
```

看到没，编译器推导出这个闭包是`FnOnce`类型的，因为闭包最后返回了`a`，交还了所有权，是不能再运行第二次了，因为闭包不再是`a`的所有者。

而`Fn/FnMut`是被认定可以多次运行的，如果交还了捕获变量的所有权，则下次就不能运行了，所以会报出前面那个错误。

## 结语

因为闭包和`rust`里的生命周期，所有权紧紧联系在一起了，有时候不怎么好理解，但多写写代码，多试几次，大概就能明白这三者之间的区别。

总之，闭包是`rust`里非常好用的功能，能让代码变得简洁优雅，是值得去学习和掌握的！
