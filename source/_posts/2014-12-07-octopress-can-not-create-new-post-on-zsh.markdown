---
layout: post
title: "Octopress Can Not Create new Post on Zsh"
date: 2014-12-07 14:44:12 +0800
comments: true
categories: [效率]
tags: [zsh,shell, octopress]
keywords: zsh, octopress, shell,no matches , widcard, rake new_post, 通配符
---

在**zsh**中执行rake new_post["Title"]时会报错，提示"no matches found"。

原因是**zsh**中若出现下列符合，则将识别为查找文件名的通配符。

 > ‘*’, ‘(’, ‘|’, ‘<’, ‘[’, or ‘?’

很不幸的时，我们在octopress中创建新blog的命令就出现了“[”这个符号。

报错信息如下：

```
➜  tonydeng.github.io git:(source) rake new_post["title"]
zsh: no matches found: new_post[title]
```

<!-- more -->

解决方法有两个：

1、 快速解决法，将rake之后都用双引号“”括起来：
```
➜  tonydeng.github.io git:(source) rake "new_post[title]"
```

2、 彻底解决法，在执行rake时，取消zsh的通配(GLOB)：
```
➜  tonydeng.github.io git:(source) echo "alias rake='noglob rake'" >> ~/.zshrc
```

### 参考：

oh-my-zsh的[issue #443](https://github.com/robbyrussell/oh-my-zsh/issues/433)

Mike Ballou的文章 [ZSH and Rake Parameters](http://mikeballou.com/blog/2011/07/18/zsh-and-rake-parameters/)
