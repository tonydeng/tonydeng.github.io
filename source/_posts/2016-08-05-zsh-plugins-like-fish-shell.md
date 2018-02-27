layout: post
title: 为ZSH实现Fish Shell的效果
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [zsh,autosuggestions,color]
date: 2016-08-05 14:26:00
keywords: zsh fish shell autosuggestions color
---

![](/images/blog/zsh-like-fish-shell/fish-shell.jpg)

很久之前就见过同事用过[`Fish Shell`](https://fishshell.com)，看到几个非常棒的特性和效果，比如下面两个特性就非常吸引我。

<!-- more -->
## Fish Shell的炫酷

![智能提示](https://fishshell.com/assets/img/screenshots/autosuggestion.png)

![语法高亮](https://fishshell.com/assets/img/screenshots/colors.png)


`Fish Shell`的**智能提示**和**语法高亮**，是我觉得非常酷炫的功能，让我眼馋，为此我也试用过多次`Fish Shell`，但是每次都坚持不了多久，因为还是有很多地方不习惯：

* 无插件系统，功能上还是比[`Oh My ZSH`](http://ohmyz.sh/)少了很多
* 不兼容`bash`语法，导致我之前的很多脚本无法运行

> [`Oh My ZSH`](http://ohmyz.sh/)才是我的真爱！


## Oh My ZSH

那么问题来了，`oh-my-zsh`中有没有插件可以实现类似的功能？

我先是在oh-my-zsh官方插件库里找了一下，但是没找到，后来发现了这样一个项目：

> zsh-users

上面的介绍说是：`Zsh community projects`，感觉是非官方的项目。

里面有两个插件：

1. [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)
1. [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

### 安装插件

这两个插件安装起来非常简单。

创建一个目录。

```bash
mkdir -p ~/.zsh/plugins
```

clone这两个项目到刚刚创建的目录下。

```bash
git clone git://github.com/zsh-users/zsh-autosuggestions.git ~/.zsh/plugins/zsh-autosuggestions
git clone git@github.com:zsh-users/zsh-syntax-highlighting.git ~/.zsh/plugins/zsh-syntax-highlighting
```

设置`.zshrc`中的`$ZSH_CUTOM`变量

```bash
# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder
ZSH_CUSTOM=~/.zsh
```

添加插件配置

```bash
plugins=(zsh-autosuggestions zsh-syntax-highlighting)
```

最终效果图如下：

![autosuggestions](/images/blog/zsh-like-fish-shell/autosuggestions.png)

途中可以看到git是绿色的，代表存在这个命令，如果打错了，它就是红色的：

![color](/images/blog/zsh-like-fish-shell/color.png)

一目了然，不用等出错了再去修正错误了。
