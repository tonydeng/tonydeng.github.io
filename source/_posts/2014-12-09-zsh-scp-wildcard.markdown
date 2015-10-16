---
layout: post
title: "zsh在scp时不能使用通配符的原因和解决方案"
date: 2014-12-09 12:01:38 +0800
comments: true
categories: [zsh, shell]
categories: [zsh, shell]
keywords: zsh, scp, shell,no matches , widcard, 通配符
description: zsh在scp时不能使用通配符的原因和解决方案

---

scp是我们经常使用的一个本地与远程服务器相互cp数据的命令，zsh是我最喜欢的shell，但是在zsh下使用scp来cp远程服务器的文件时，却出现这样的错误。

```
scp ip:/home/tonydeng/logs/*.log .
zsh: no matches found: ip:/home/tonydeng/logs/*.log
```

同样地命令，在bash下确实可以执行的，这个原因是什么呢？

由于zsh不会按照远程地址上的文件去扩展参数，当你使用```ip:/home/tonydeng/logs/*.log```，因为本地当前目录中，是不存在```ip:/home/tonydeng/logs/*.log```，所以匹配失败。默认情况下，bash 在匹配失败时就使用原来的内容，zsh 则报告一个```no matches```的错误。

在zsh中执行```setopt nonomatch```，告诉它不要报告```no matches```的错误，而是当匹配失败时直接使用原来的内容。

实际上，不管是 bash 还是 zsh，不管设置了什么选项，只要把```ip:/home/tonydeng/logs/*.log```加上引号，如```"ip:/home/tonydeng/logs/*.log"```，就可解决问题。

当然根本的解决办法还是告诉zsh不要报告```no matches```错误。

建议你执行下面的命令一劳永逸，好好享受zsh给你带来的快乐和便利吧。
```
echo "setopt nonomatch" >> ~/.zshrc
```
或
```
echo "set -o nonomatch" >> ~/.zshrc
```


其实我之前的[这篇blog](/blog/2014/12/07/octopress-can-not-create-new-post-on-zsh/)也是同样的原因。

参考

stackoverflow的问答[Why zsh tries to expand * and bash does not?](http://stackoverflow.com/questions/20037364/why-zsh-tries-to-expand-and-bash-does-not)
