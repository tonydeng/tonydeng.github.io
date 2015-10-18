---
layout: post
title: "Brew Update失败处理方法"
date: 2014-12-31 00:21:39 +0800
published: true
comments: true
categories: [Mac OSX]
tags: [brew, mac osx] 
keywords: mac osx brew update error
description: brew update失败处理方法
---

使用```brew update```时报错，错误如下：

```
error: Your local changes to the following files would be overwritten by merge:
```

我看了diff，代码变更和网站上的代码变更一致，但是版本控制没有跟踪到。

解决办法就是同步到最新的版本，或者如果有自行修改或者第三方的修改要保留，可以merge

方法1.删除这些更改的文件，然后```brew update```

方法2.重新获取最新的文件，并且```hardreset```

```
git reset --hard FETCH_HEAD
```

或者

```
cd /usr/local && sudo git reset --hard FETCH_HEAD
```


```
cd `brew --prefix`
git remote add origin https://github.com/mxcl/homebrew.git
git fetch origin
git reset --hard origin/master
```

方法3.保留更改，brew update，再应用更改

```
You have made some changes in the brew repository.

You can either do a git stash, brew update, git stash pop to keep your changes, or git reset –hard HEAD, brew update to reset the changes.
```

接着brew doctor又提示

```
Warning: The /usr/local directory is not writable.
Even if this directory was writable when you installed Homebrew, other
software may change permissions on this directory. Some versions of the
“InstantOn” component of Airfoil are known to do this.
```
是group的问题，默认group是wheel，brew要求是admin
view source

```
sudo chgrp -R admin /usr/local
```
