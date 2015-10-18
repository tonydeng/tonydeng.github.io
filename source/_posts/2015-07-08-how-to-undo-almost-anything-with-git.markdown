---
layout: post
title: "Git的各种Undo技巧"
date: 2015-07-08 17:04:12 +0800
published: true
comments: true
categories: [效率]
tags: [git, undo]
keywords: git undo
description: Git各种Undo技巧
---

![git undo](/images/blog/git-undo/git-undo.jpg)

[GitHub](https://github.com)的[How to undo (almost) anything with Git](https://github.com/blog/2019-how-to-undo-almost-anything-with-git)这篇文章介绍了[Git](https://git-scm.com)使用中的各种Undo技巧。


任何版本控制系统中最有用的功能之一就是能够**"撤销（undo）"**你之前的错误。在Git中**“undo”**功能可能因为场景的不同而有些许的差异。

当你进行一个新的提交时，Git会保存你在这个特定时间点的快照到本地的仓库中，之后，你可以通过Git来回到你早期的某个版本。

我们来先看看一些需要你“撤销”的常见场景，你可以尝试使用Git来用最佳的方式来解决它。

## 撤销已经推送到远程的变更

#### 场景：

你已经执行**git push**,把你的修改推送到远程的仓库，现在你意识到之前推送的**commit**中有一个有些错误，想要撤销该commit。

#### 方案：

```
git revert <SHA>
```

#### 原理：

**git revert** 会创建一个新的**commit**，它和指定SHA对应的**commit**是相反的（或者说是反转的）。如果原型的commit是“物质”，那么新的**commit**就是“反物质”。

任何从原来的commit里删除的内容都会再新的**commit**里被加回去，任何原来的**commit**中加入的内容都会在新的commit里被删除。

这是Git中最安全、最基本的撤销场景，因为它并不会改变历史。所以你现在可以**git push**新的**“反转”commit**来抵消你错误提交的commit。

## 修正最后一个commit的消息

#### 场景：

你在最后一条commit消息里有一个笔误，已经执行了 **git commit -m 'Fixes bug #42'** ,但是在git push之前你意识到这个消息应该是**"Fix bug #43"**。

#### 方法

你可以使用下面的命令：

```
git commit --amend
```
或

```
git commit --amend -m 'Fixes bug #43'
```

#### 原理：

**git commit --amend** 会用一个新的commit更新并替换最近的**commit**，这个新的**commit**会把任何修改内容和上一个**commit**的内容结合起来。如果当前没有提出任何修改，这个操作就只会把上次的**commit**重写一遍。

## 撤销“本地的”修改

#### 场景：

一只喵从键盘上走过（在我们家就是儿子小手在键盘上划拉），无意中保存了修改，然后破坏了编辑器。不过，你还没有**commit**这些修改。你想要恢复被修改文件里的所有内容--就像上次**commit**的时候一模一样。

#### 方法：

```
git checkout -- <bad filename>
```

#### 原理

`git checkout`会把工作目录中的文件修改到Git之前记录的某个状态。你可以提供你想返回的分支或者特定的SHA，或者在缺省情况下，GIt会认为你希望checkout的是**HEAD**，当前checkout分支的**最后一次commit**。

记住： **你用这种方法“撤销”的任何修改真的会完全消失**。因为它们从来没有被提交过，所以之后Git也无法帮助我们恢复它们。你一定要确保自己了解在这个操作中丢掉的东西是什么？（也行可以利用`git diff`先确认一下）

## 重置“本地的”修改

#### 场景：

你在本地提交了一下东西（还没有push），但是所有这些东西都很糟糕，你希望撤销前面的三次提交（就像它们从来没有发生过一样）。

#### 方法：

```
git reset <last good SHA>
```

或

```
git reset --hard <last good SHA>
```

#### 原理：

`git reset`会把你的代码库历史返回到指定的SHA状态。这样就像是这些提交从来没有发生过。缺省情况下，`git reset`会保留工作目录。这样，提交是没有了，但是修改内容还在磁盘上。这是一种安全的选择，但通常我们会希望一步就“撤销”提交已经修改内容(这就是`--hard`选项的功能)。

## 在撤销“本地修改”之后再恢复

#### 场景：

你提交了几个commit，然后用`git reset --hard`撤销了这些修改（见上一段），接着你又意识到：你希望还原这些修改！

#### 方法：

```
git reflog
```
和

```
git reset
```
或

```
git checkout
```

#### 原理：

`git reflog`对于恢复项目历史是一个超棒的方式。你可以恢复几乎任何（commit过的）东西。

你可以能熟悉`git log`命令，它会显示commit的列表。 git reflog也是类似的，不过它显示的是一个HEAD发生改变的时间列表。

一些注意事项：

* 它涉及的只是HEAD的改变。在你切换分支、用git commit进行提交、以及用git reset撤销commit时，HEAD会发生改变，但当你使用`git checkout -- <bad filename>`撤销时，HEAD并不会发生改变。就像我们在上面说的，这些修改从来没有被提交过，因此reflog也无法帮助我们恢复它们。
* `git reflog`不会永远保持。Git会定期清理那些“用不到的”对象。不要指望几个月前的提交还一直躺着那里。
* 你的reflog就是你的，只是你的，你不能用`git reflog`来恢复另外一个开发者没有push过的commit。

![git reflog](/images/blog/git-undo/reflog.png)

那么...你如何来利用reflog来“恢复”之前“撤销”的commit呢？它取决于你想做到的到底是什么。

* 如果你希望准确的恢复项目的历史到某个时间点，用`git reset --hard <SHA>`
* 如果你希望重建工作目录里的一个或多个文件，让它们恢复到某个时间点的状态，用`git checkout <SHA> -- <filename>`
* 如果你希望把这些commit里的某个重新提交到你的代码库里，用`git cherry-pick <SHA>`

## 利用分支的另一种做法

#### 场景：

你进行了一些提交，然后意识到你开始checkout的是master分支。希望这些提交进入到另外一个特性（feature）分支。

#### 方法：

```
git branch feature
git reset --hard origin/master
git checkout feature
```

#### 原理：

你可能习惯用 `git checkout -b <name> `创建一个新的分支（这是创建新分支并马上checkout的流行捷径），但是你不希望马上切换分支。这里，`git branch feature`创建了一个叫做feature的新分支，并指向你最近的commit，但是还是让你checkout在master分支上。

下一步，在提交任何新的commit之前，用`git reset --hard` 把master分支倒回origin/master。不过别担心，那些commit还在feature分支里。

最后，用`git checkout `切换到新的feature分支，并让你最近所有的工作成果都完好无损。

## 及时分支，省去繁琐

#### 场景：

你在master分支的基础上创建一个feature分支，但是master分支已经滞后于origin/master很多。现在master分支已经和origin/master同步，你希望在feature上的提交从现在开始，而不是从滞后很多的地方开始。

#### 方法：

```
git checkout feature
```

和

```
git rebase master
```

#### 原理：

要达到这个效果，你本来可以通过`git reset`（不加`--hard`，这样可以再磁盘上保留修改）和`git checkout -b <new branch name>`然后再重新提交修改，不过这样做的话，你就失去提交历史。我们有更好的办法。

`git rebase master`会做下面的这些事情：

1. 首先它会找到你当前checkout的分支和master分支的共同祖先。
2. 然后它reset当前checkout的分支到那个共同祖先 ，在一个临时保存区存放所有之前的提交。
3. 然后它把当前checkout的分支提到master的末尾部分，并从临时保存区重新把存放的commit提交到master分支的最后 一个commit之后。

## 大量的撤销/恢复

#### 场景：

你向某个方向开始实现一个特性，但是你半路意识到另一个方案更好。你已经进行了十几次的提交，但是你现在只需要其中的一部分。你希望其他不需要的提交统统消失。

#### 方案：

```
git rebase -i <carlier SHA>
```

#### 原理：
`-i` 参数会让rebase进入“交互模式”。它开始类似于签名讨论的rebase，但在重新进行任何提交之前，它会暂停下来并允许你详细的修改每一个提交。

`rebase -i` 会打开你缺省的文本编辑器，里面列出候选的提交。

![rebase -i](/images/blog/git-undo/rebase_1.png)

前面两列是键： 第一个是选定的命令，对应第二列里的SHA确定的commit。缺省情况下，`rebase -i`假定每个commit都要通过pick命令被运行。

要丢弃一个commit，只要在编辑器中删除那一行就行了。如果你不再需要项目里面的那几个错误的提交， 删除你想删除的几行。

如果你需要暴漏commit的内容，而是对commit消息进行编辑，你可以使用reword命令，将第一列的pick替换成reword（或者直接使用r）。有人会觉得再这里直接重写commit消息就行了，但是这样是不管用的（rebase -i会忽略SHA列前面的任何东西，它后面的文本只是用来帮助我们记住那一串SHA代表什么）。当你完成rebase -i的操作之后，你会被提示输入需要编写的任何commit消息。

如果你需要把两个commit合并在一起，你可以使用squash或者fixup命令。

![rebase -i](/images/blog/git-undo/rebase_edit.png)

squash和fixup会“向上”合并（带有这两个命令的commit会被合并到他的签一个commit里）。上面的这个例子里，`0835fe2`和`6943e85`会合并成一个commit，`38f5c4c`和`af67f82`会被合并成另一个。

如果你选择了squash，Git会提示我们给新合并的commit一个新的commit消息； fixup则会把合并清单里的第一个commit的消息直接给新合并的commit。


在你保存并推出编辑器的时候，Git会按从顶部到底部的顺序运用你的commit。你可以通过在保存前修改commit顺序来改变运用的顺序。如果你愿意，你也可以同如下安排把`af67f82`和`0835fe2`合并到一起。

![rebase -i](/images/blog/git-undo/rebase_fixup.png)

## 修复更早期的commit

#### 场景：

你在一个更早期的commit里忘记了加入一个文件，如果更早的commit能包含这个忘记的文件就太棒了。你还没有push，但这个commit不是最近的，所以你还没法用commit --amend

#### 方法：
```
git commit --squash <SHA of the earlier commit>
```
和

```
git rebase --autosquash -i <even earlier SHA>
```

#### 原理：

`git commit --squash `会创建一个新的commit，它带有一个commit消息，类似于squash! Earlier commit。(你也可以手工创建一个带有类似commit消息的commit，但是 commit --squash 可以帮你省下输入的工作)

如果你不想被提示为新合并的commit输入一条新的commit消息，你也可以利用 `git commit --fixup `。在这个情况下，你很有可能会用`commit --fixup`，因为你只是希望在rebase的适合，使用早期commit的commit消息。

`rebase --autosquash -i` 会激活一个交互式的rebase编辑器，但是编辑器打开的适合，在commit清单里任何squash!和fixup！的commit都已经配对到目标commit上了。

![rebase --autosquash -i](/images/blog/git-undo/rebase_autosquash.png)

在使用 `--squash` 和 `--fixup`的适合，你可能不记得想要修正的commit的SHA了（只是记得它是前面的第1个或者第5个commit）。你会发现Git的`^`和`~`操作符特别好用。`HEAD~`是HEAD的前一个commit。`HEAD~4`是HEAD往前第4个（或者一起算，倒数第5个commit）。

## 停止追踪一个文件

#### 场景：

你偶然把application.log加到代码库里了，现在每次你运行应用，Git都会报告在application.log里有未提交的修改。你把 *.log放到了.gitignore文件里，可文件还是在代码库里，你怎样才能让Git“撤销”对这个文件的追踪呢？

#### 方法：

```
git rm --cached application.log
```

#### 原理:

虽然.gitignore会阻止Git追踪文件的修改，甚至不关注文件是否存在，但这只是针对那些以前从来没有追踪过的文件。一旦有个文件被加入并提交了，Git就会持续关注改文件的改变。类似地，如果你利用`git add -f`来强制或覆盖了.gitignore，Git还会持续追踪改变的情况。之后你就不必用-f来添加这个文件了。

如果你希望从Git的追踪对象中删除那个本应忽略的文件，`git rm --cached`会从追踪对象中删除它，但让文件在磁盘上保持原封不动。因为现在它已经被忽略了，你在`git status`里就不会再看见这个文件，也不回再偶然提交该文件的修改了。

这就是如何在Git里撤销任何操作的方法。要了解更多关于本文中用到的Git命令，请查看下面的相关文档。

* [checkout](http://git-scm.com/docs/git-checkout)
* [commit](http://git-scm.com/docs/git-commit)
* [rebase](http://git-scm.com/docs/git-rebase)
* [reflog](http://git-scm.com/docs/git-reflog)
* [reset](http://git-scm.com/docs/git-reset)
* [revert](http://git-scm.com/docs/git-revert)
* [rm](http://git-scm.com/docs/git-rm)
