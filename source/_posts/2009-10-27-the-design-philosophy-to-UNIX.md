layout: post
title: UNIX设计哲学
published: true
comments: true
sharing: true
footer: true
categories: [程序设计]
tags: [linux, unix, design, engineered]
date: 2009-10-27 21:59:06
keywords: linux, unix, 设计哲学
---

![KISS](/images/blog/kiss.png)

现在所有常用的操作系统基本上都是在Unix的基础上衍生--Mac OSX、Linux（最少是借鉴--Windows）出来的，而且基本上大学的操作系统的课程也是拿Unix系统来讲解的。

<!--more-->
最近在看[《Unix 编程艺术》](http://book.douban.com/subject/1467587/)这本书，上网搜索了相关的信息，发现好多人都在谈论“UNIX设计哲学”。[Wikipedia](http://en.wikipedia.org/wiki/Unix_philosophy#cite_note-0)上也列出了好几个版本，不同的人有不同的总结。

不过，这几个版本中所有人都同意** “简单原则”--尽量用简单的方法解决问题--是“Unix哲学”的根本原则 **。这也就是著名的`KISS(keep it simple,stupid)`，意思是“保持简单和笨拙”。关于KIIS大家可以看[阮一峰大大的Blog](http://www.ruanyifeng.com/blog/2009/06/unix_philosophy.html)

Unix哲学中更多的内容不是这些先哲们口头表述出来的，而是由他们所作的一切和Unix本身所作出的榜样体现出来的。

从整体上来说，可以概括为以下几点：

1. 模块原则：使用简洁的接口拼合简单的部件。

2. 清晰原则：清晰胜于机巧。

3. 组合原则：设计时考虑拼接组合。

4. 分离原则：策略同机制分离，接口同引擎分离。

5. 简洁原则：设计要简洁，复杂度能低则低。

6. 吝啬原则：除非确无他法，不要编写庞大的程序。

7. 透明性原则：设计要可见，以便审查和调试。

8. 健壮原则：健壮源于透明与简洁。

9. 表示原则：把知识叠入数据以求逻辑质朴而健壮。

10. 通俗原则：接口设计避免标新立异。

11. 缄默原则：如果一个程序没什么好说的，就沉默。

12. 补救原则：出现异常时，马上退出并给出足够错误信息。

13. 经济原则：宁花机器一分，不花程序员一秒。

14. 生成原则：避免手工hack，尽量编写程序去生成程序。


