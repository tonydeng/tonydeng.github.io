---
layout: post
title: "效率脚本:删除指定项目的所有feature分支"
date: 2014-12-08 11:14:02 +0800
comments: true
categories: [shell, git]
tags: [shell, git]
keywords: shell, git, gitflow, shell script，效率脚本
description: 效率脚本:删除指定项目的所有feature分支 由于我们基于gitlab来进行code review，所有的分支都会push到远程仓库来进行code review。当一个版本完成之后，在远程和本地都会存在一些过期的、无用的分支。对于有些强迫症的我来说，保留这些无用的分支实在很难受，一个一个手工的删除这些分支也不符合我的风格，还是写个脚本来批量处理吧。
---

使用Git来管理代码，的确给我们带来很多方便，尤其是使用gitflow流程，让我们开发流程更加清晰，响应需求变化速度更快。

由于我们基于gitlab来进行code review，所有的分支都会push到远程仓库来进行code review。当一个版本完成之后，在远程和本地都会存在一些过期的、无用的分支。对于有些强迫症的我来说，保留这些无用的分支实在很难受，一个一个手工的删除这些分支也不符合我的风格，还是写个脚本来批量处理吧。

## 删除哪些分支？

由于大部分的过期的分支都是feature分支，所以我们就来这些已经合并过的本地和远程的feature分支吧。

### Shell脚本如下：
``` bash
#!/bin/bash
WORKSPACE=$1
echo $WORKSPACE

cd $WORKSPACE

BRANCHS=`git branch -r|awk -F '/' '{if($2=="feature")print $2"/"$3"/"$4}'`
for br in $BRANCHS
do
        local_result=`git branch -d $br`
        echo "git branch -d $br result: $local_result"
        remote_result=`git push origin :$br`
        echo "git push origin :$br result: $remote_result"
done
echo $BRANCHS
```

### 执行方法
``` bash
~/sh/git/feature_branch_del.sh your_git_project
```
