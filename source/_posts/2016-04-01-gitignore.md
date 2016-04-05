layout: post
title: gitignore最佳实践
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [git, gitignore]
date: 2016-04-01 16:00:17
keywords: git gitignore 最佳实践
---

![gitignore](/images/blog/gitignore.jpg)

## gitignore的使用场景

使用Git的同学都知道`.gitignore` 配置文件用于配置不需要加入版本管理的文件，对版本管理带来很大的便利。今天有个需求就是忽略版本库下除少数几个文件和文件夹之外的所有文件，首先想到的方式是使用gitignore树的概念，即在需要的文件夹下都添加 `.gitignore` 文件，并在其中设定相应的规则。但是，这种方式比较麻烦。

## gitignore的使用模式

好好研究了一下gitignore的语法，知道了`.gitignore` 文件过滤有两种模式：开放模式和保守模式。

### 开放模式

开放模式负责设置过滤哪些文件和文件夹

例如： 


 	/target/ 表示项目根目录下的target文件夹里面所有的内容都会被过滤，不被跟踪
 	
 	.classpath 表示项目根目录下的.classpath文件会被过滤，不被跟踪

### 保守模式

保守模式负责设置哪些文件不被过滤，也就是哪些文件要被跟踪

例如：

 	!/target/*.h 表示target文件夹目录下所有的.h文件将被跟踪

还有就是，gitignore是从上到下逐行匹配的，因此.gitignore文件的编写原则就是：

**先编写开放模式，在编写保守模式**


要不然，开放模式的规则会把保守模式的规则给覆盖了。

## 例子

下面附上我的 `.gitignore` 文件的示例：

```
/*
!.gitignore
!/posts/
```

只跟踪版本库中的.gitignore文件和posts目录。这里需要注意的是：

	一定是/*而不是*，/*表示当前目录下的所有文件，而不是所有文件*
	
## gitignore的简单语法：

1. 以斜杠“/”开头表目录

2. 以星号“*”通配多字符

3. 以问号“?”通配单字符

4. 以方括号“[]”包含单个字符的匹列表

5. 以叹号“!”对匹配结果反

## 关于

附上几个对大家有帮助的gitignore相关的链接。

[git-scm官网的gitinore文档](https://git-scm.com/docs/gitignore)

[progit的Git基础中gitignore相关的部分](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%AE%B0%E5%BD%95%E6%AF%8F%E6%AC%A1%E6%9B%B4%E6%96%B0%E5%88%B0%E4%BB%93%E5%BA%93#%E5%BF%BD%E7%95%A5%E6%96%87%E4%BB%B6)

[Github的Ignoring files相关文章](https://help.github.com/articles/ignoring-files/)

[我forked的Github的gitignore项目](https://github.com/tonydeng/gitignore)

[自动生成gitignore配置的网站](https://www.gitignore.io/)