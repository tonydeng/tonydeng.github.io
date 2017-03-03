layout: post
title: Python的JIT
published: true
comments: true
sharing: true
footer: true
categories: [性能]
tags: [python,JIT]
date: 2017-03-03 13:37:38
keywords: Python JIT
---

## 编译型 or 解释型？
玩过 `Python` 的同学应该都听过 `python` 是解释型语言，那么什么是解释型语言，其实在现代的编程语言中，关于解释型 和 编译型 的界限越来越模糊了，这里我根据我自己的理解下个定义：

* 解释型：不能直接编译成可执行二进制程序的语言
* 编译型：可以直接编译成可执行二进制程序运行的语言

根据这个定义的话，可以明确给出界定，`C/C++` 是 编译型 的，而 `Java` 则是 解释型 的，为什么我没说 `Python，因为` Python 我也不敢说它是哪种类型的。为什么？因为这要看你说的 `Python` 是什么？是说 `Python` 编程语言，还是 `CPython` 解释器。如果说的是 `CPython` 解释器，那么毫无疑问 `Python` 是解释型的；那如果说的是编程语言，那么还真无法界定，因为 `Python` 规范只定义了 `Python` 的语法规定，并没有规定要如何实现编译解析，所以无法确定。
<!-- more -->
![python is a programming language](https://ooo.0o0.ooo/2017/02/04/58959aaa98af7.jpg)

那么是否就可以说解释型语言一无是处呢？那肯定不是的，存在即合理，虽然解释型语言的速度相比编译型会慢很多，但是解释型语言具有一些编译型无法比拟的优点：

1. 一次编译，处处执行(Java的最大卖点)
1. 可以确保你看到的代码就是你执行的代码(python)
1. 弱类型，开发速度很快

## Python 执行过程

当然，这里扯那么多其实都不是关键，但是，关键是我这里的观点是 Python 是解释型的，而且我所说的 `Python` 是指 `CPython` 解释器器执行的 `Python` 语言代码。那么这里的问题就来了，`CPython` 解释器执行 `Python` 代码的时候是直接将原始的 `Python` 代码加载进行逐条执行么？可以说无论是 `Python2` 还是 `Python3` 都不是，原始的 `Python` 代码都会被编译成字节码，然后再执行。但是新版本的 `Python` 和旧版本的 `Python` 有一些不一样，旧版本的 `Python` 是在内存中编译成字节码，而新版本的 `Python` 则会编译成 `pyc` 文件，放置在源文件的目录。用一张图来表示就是这样的：

![python build source code](https://ooo.0o0.ooo/2017/02/04/58959ffd5678f.png)

那么这里就有一个问题了，为什么不直接执行 `Python` 代码呢，而是要编译一遍转换成字节码？其实很多人也想得到，其实就想加快一下 `Python` 的执行速度，因为边执行边编译速度肯定比先编译好快的，尤其是 `CPU` 资源占用多的情况更是如此；同时，先编译的话我们可以有更好得优化空间，我们可以做一些更好得代码优化，例如这样的代码：

```python3
a = 1
b = 2
a = a + b
```

如果一边编译一边执行，我们就需要执行 3 条语句了，如果我们预先编译的话，完全可以优化成这样：

```Python3
a = 3
```

就是这么好的效果。随着人们的不断使用，人们觉得仅仅编译成字节码的速度不够快，因为毕竟不是机器码，于是就有人想把 `python` 从源码编译成二进制可执行程序的，不是没有，有！现在有很多工具可以将 Python 代码编译成可执行程序，例如：`Py2exe`、`Installer` 和 `freeze`。正如前面所说，编译成二进制代码是不错，但是缺抛弃了可迁移性，就不能一次编写，处处执行了。

##JIT(just-in-time)

为了兼具移植性和性能，聪明的工程师们发明了 `JIT` 这个东西，所谓的 `JIT` 就是说在解释型语言中，对于经常用到的或者说有较大性能提升的代码在解释的时候编译成机器码，其他一次性或者说没有太大性能提升的代码还是以字节码的方式执行。这样的话，就能在保证移植性的同时，又能让性能提升一大截，这里学问其实是很深的，因为我们怎么知道编译哪段代码能让我们的性能提升一大截呢？最简单的方式就是看循环，循环体中的代码都给他编译的，这样的话，大部分情况下我们的代码都是比原来高效的，但是，这也仅仅提高了循环的速度，还有其他很多代码有提升的可能，关于这个问题，这里也无法详说，有兴趣的同学可以自行搜索学习。

继续以 Python 为例介绍一下 `Python` 中的 `JIT`，在 `CPython` 中是没有 `JIT` 这个功能的，但是，有一个扩展模块 `Psyco` 可以实现这个功能；除此之外，其他的 `python` 解释器实现自带 `JIT`，例如 `pypy`。这里也不讨论如何使用 `Psyco` 加速 `Python` 代码了，有兴趣的同学可以看下 [Charming Python: Make Python run as fast as C with Psyco](https://www.ibm.com/developerworks/library/l-psyco/)

## 小结

本文从简单的介绍 `Python` 代码的执行过程出发，探讨了一下解释型和编译型程序的区别，然后引出 JIT 的必要性和强大。然而在生产中 `JIT` 并不会太多用到，因为目前 `Python` 还未被大规模应用于高并发的互联网应用中，或者说是性能要求高的场景。然而当真的需要面对这些场景的时候，难道直接编译 `Python` 代码不会是更好的选择？


## Reference

[深入浅出 JIT 编译器（java）](https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/)
[How Python Runs Programs](http://www.devshed.com/c/a/python/how-python-runs-programs/)
[If Python is interpreted, what are .pyc files?](http://stackoverflow.com/questions/2998215/if-python-is-interpreted-what-are-pyc-files)
[Charming Python: Make Python run as fast as C with Psyco](https://www.ibm.com/developerworks/library/l-psyco/)
[说说Python程序的执行过程](http://www.cnblogs.com/kym/archive/2012/05/14/2498728.html)
