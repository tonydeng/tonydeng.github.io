layout: post
title: Git常见分支管理实践
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [git, branch, scm]
date: 2017-08-31 16:41:28
keywords: git git分支 分支管理实践
---

Git是目前最流行的代码版本管理系统，像[Github](https://github.io)也被称为全球最大的同性交友网站 ;-) 可见Git在工程师人群中的流行程度。不过在使用Git时，经常会碰到一个问题，就是采用什么样用的分支模型进行管理，Git官方也提供了很多分支模型推荐（[分布式工作流程](https://git-scm.com/book/zh/v2/%E5%88%86%E5%B8%83%E5%BC%8F-Git-%E5%88%86%E5%B8%83%E5%BC%8F%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)这篇文章有相应的记载）。

我们这次主要介绍三种常见的Git分支模型。

<!-- more -->

## 单主干

单主干的分支实践（Trunk-based development,TBD）在SVN中比较流行。[Google](http://paulhammant.com/2013/05/06/googles-scaled-trunk-based-development/)和[Facebook](http://paulhammant.com/2013/03/13/facebook-tbd-take-2/)都使用这种方式。`trunk`是SVN中主干分支的名称，对应Git中则是`master`分支。

TBD的特点是所有团队成员都在单个主干分支上进行开发。当需要发布时，先考虑使用标签（`tag`）,即`tag`某个`commit`来作为发布的版本。

如果仅仅依靠`tag`不能满足要求，则从主干分支创建发布分支。

`bug`修复在主干分支进行，再`cherry-pick`到发布分支。

![TBD中分支流程](https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/img001.png)

由于所有开发人员都在同一个分支工作，团队需要合理的分工和充分沟通来保证不同开发人员的代码尽可能少的发生冲突。因此持续集成和自动化是必要的，用来及时发现主干分支中的bug。因为主干分支是所有开发人员公用的，一个开发人员引入的bug可能对所有人造成影响。

不过好处是由于分支所带来的额外开销非常小。开发人员不用频繁在不同的分支之间切换。


![Github Flow的分支流程](https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/img002.png)

`GitHub flow`的好处在于非常简单实用。开发人员需要注意的事项非常少，很容易形成习惯。当需要进行任何修改时，总是从`master`分支创建新分支。完成之后通过`pull request`和相关的代码审查来合并回`master`分支。`GitHub flow `要求项目有完善的自动化测试、持续集成和部署等相关的基础设施。每个新分支都需要测试和部署，如果这些不能自动化进行，会增加开发人员的工作量，导致无法有效地实施该流程。这种分支实践也要求团队有代码审查的相应流程。

## Git Flow

[Git Flow](http://nvie.com/posts/a-successful-git-branching-model/)应该是目前流传最广的Git分支管理实践。

![git flow](http://nvie.com/img/git-model@2x.png)

Git Flow围绕的核心概念是版本发布（release）。因此Git Flow适用于各种版本发布周期的项目。

Git Flow流程中包含5类分支，分别是master、develop、feature（新功能分支）、release（发布分支）和 hotfix（热修复分支）。这些分支的作用和生命周期各不相同。

### 主分支
![the main branches](http://nvie.com/img/main-branches@2x.png)
主分支是`git flow`整个分支模型的核心，它们有无限的声明周期。主分支分别是`master`和`develop`分支。

`origin/master`分支应该对每个`Git`用户都很熟悉。与`master`分支并行，另一个分支称为`develop`。

我们认为`origin/master`是`HEAD`源代码生产就绪状态的主要分支。

`origin/develop`是`HEAD`源代码反映下一版本的最新发展变化的状态的主要分支。也有人会称之为“整合分支”，可以基于`develop`分支定时执行每日构建的工作。

当`develop`分支中的源代码达到一个稳定点并准备被`release`时，所有的改变应该以某种方式合并回`master`，然后用一个版本号来标记。

因此，每次将更改合并回主数据库时，这都是定义的新产品发布。在这方面往往非常严格，所以可以使用一个`Git Hook`脚本来自动构建和发布软件到生产服务器。

### 支持分支

除了主分支以外，`git flow`还定义了三类支持分支，分别用于不同场景。这些分支都是有限的声明周期，最终都是会被删除的。

这些分支分别是：
- Feature分支
- Release分支
- Hotfix分支

每个分支都有特定的目的，对于哪些分支是用于开发工作，哪些分支是其他分支的合并目标，都有严格的规定。 

分支类型只是按照我们使用它们来进行的分类。从技术的角度来看，这些分支并不“特殊”的，它们也仅仅只是普通的Git分支。


#### Feature分支
![feature branchs](http://nvie.com/img/fb@2x.png)

从上面的图就能够很清楚的知道，`feature`分支可能来源于`develop`，并且只能合并回`develop`。

`feature`分支（或有时称为`topic`分支）用于为即将到来的版本开发新功能。 

当开始一个特征的开发时，这个特征将被合并到的目标版本可能在那个时候是未知的。 feature分支的本质是，只要功能处于开发阶段，它就会存在，但最终会被合并回`develop`（为了明确地添加新功能到即将发布的版本）或丢弃（如果是令人失望的实验）。


#### Release分支

![release branches](/images/blog/git-flow-release-branchs.jpg)

`release`分支可能来源于`develop`，并且只能合并回`develop`和`master`。

`release`分支主要用来产品发布的准备。`release`分支允许有一些小的bug修复和准备发表的一些配置或元数据(版本号，生成日期等)的修改和提交，在发表版本之前，在`release`分支上完成这些所有的工作。此时，`develop`可以继续接受下一个大版本的feature分支的合并。

在`release`分支建立之后，即将发行的版本才被分配了一个版本号。在创建`release`分支之前，`develop`分支只是反映了“下一个版本”将要有的变化，但是不知道这个“下一个版本”是否最终会变成`0.3`或`1.0`，直到`release`分支开始创建才能确定。正常来说，这个版本号确定的决定是在`release`分支开始的时候做出的，并且需要遵循项目的版本号规则。

#### Hotfix分支

![hotfix beanches](http://nvie.com/img/hotfix-branches@2x.png)

`hotfix`分支可能来源于`master`，并且只能合并回`develop`和`master`。

`hotfix`分支非常类似于`release`分支，因为它也意味着准备新的产品发布，尽管是无计划的。

 `hotfix`分支是由于需要立即修复生产版本的`bug`而产生的。 当生产版本中的关键错误必须立即解决时，将会基于`master`来创建一个新的`hotfix`分支出来。

这时团队成员（开发部门）的工作可以在`develop`、`feature`、`release`分支中继续，而另一个人则在`hotfix`分支中解决生产环境的bug，紧急修复并上线发布。

## Github Flow

[Github Flow](http://scottchacon.com/2011/08/31/github-flow.html)是[Github](https://github.com)所使用的一种流程，它主要是依托于`git-flow`，并依托Github的`pull request`功能创建的分支模型和流程。

在`Github Flow`中，`master`分支中也是代表着稳定的代码。该分支已经或即将被部署在生产环境。

![pull requests](https://cloud.githubusercontent.com/assets/70/6769770/61a2dcba-d0a8-11e4-9924-3576232053ee.png)

`master`分支的作用是提供一个稳定可靠的代码基础。任何开发人员都不允许把未测试或未审核的代码直接提交到`master`分支。

对于代码的任何修改，包括`bug`修复，`hotfix`、新功能开发都在单独的分支中进行。不管是一行代码的小改动，还是需要几个星期开发的新功能，都采用同样的方式来管理。当需要进行修改时，从`master`创建一个新的分支。新分支的名称应该简单清晰的描述该分支的作用。所有相关的代码修改都在新分支中进行。开发人员可以自由的提交代码和`push`到远程仓库。

当新分支中的代码全部完成之后，通过`Github`提交一个新的`pull request`。团队中的其他人会对代码进行审核，提出相关修改意见。由持续集成服务器（如`Jenkins`）对新分支进行自动化测试。当代码通过自动化测试和代码审核之后，该分支的代码被合并到`master`分支。然后从`master`分支不是到生产环境。

