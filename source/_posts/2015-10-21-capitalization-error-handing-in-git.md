layout: post
title: Git中文件名大小写引起的错误处理
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [git]
date: 2015-10-21 17:02:00
keywords: capitalization error handing in git
---
![git branching](/images/blog/git-branching.png)

在团队使用Git的时候，尤其是多人合作的项目，经常会出现一个问题，就是由于同一个文件名大小写不一致导致无法合并的问题。

那我们应该怎么来解决呢?

<!-- more -->

可以使用git rm --cached将冲突的文件从Git仓库的缓存中删除，然后改名后再加入到git中

```
git rm --cached <filename>

mv <old_filename> <new_filename>

git add <new_filename>

git commit -m 'rename <new_filename>' <new_filename>
```

当然，为了一劳永逸，我们可以让团队成员都更改配置git的大小写敏感，避免某些windows用户继续制造这样的问题。

```
git config core.ignorecase false
```