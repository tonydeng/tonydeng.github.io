layout: post
title: Linux文件的特殊权限
published: false
comments: true
sharing: true
footer: true
categories: [效率]
tags: [linux, setuid, setgid, sticky bit]
date: 2015-11-10 22:35:14
keywords: linux 特殊权限
---
# Linux文件的普通权限


# Linux特殊权限

每一个文件有所有者及组编号，set uid ；set gid可以改变用户对文件具有的权限：写和执行.
 
1. setuid: 在执行时具有文件所有者的权限.
2. setgid: 设置目录. 一个目录被标上setgid位,此目录下创建的文件继承该目录的属性.
3. sticky bit: 该位可以理解为防删除位. 设置sticky bit位后，就算用户对目录具有写权限，但也只能添加文件而不能删除文件。


如何设置：

操作这些标志与操作文件权限的命令是一样的, 都是 chmod. 有两种方法来操作,
1) chmod u+s temp -- 为temp文件加上setuid标志. (setuid 只对文件有效，U=用户)
chmod g+s tempdir -- 为tempdir目录加上setgid标志 (setgid 只对目录有效，g=组名)
chmod o+t temp -- 为temp文件加上sticky标志 (sticky只对文件有效)

2) 采用八进制方式. 这一组八进制数字三位的意义如下,
abc
a - setuid位, 如果该位为1, 则表示设置setuid
b - setgid位, 如果该位为1, 则表示设置setgid
c - sticky位, 如果该位为1, 则表示设置sticky

设置后, 可以用 ls -l 来查看. 如果本来在该位上有x, 则这些特殊标志显示为小写字母 (s, s, t). 否则, 显示为大写字母 (S, S, T)
如:

```
rwsrw-r-- 表示有setuid标志 (rwxrw-r--:rwsrw-r--)
rwxrwsrw- 表示有setgid标志 (rwxrwxrw-:rwxrwsrw-)
rwxrw-rwt 表示有sticky标志 (rwxrw-rwx:rwxrw-rwt) 
```

# 参考

[我07年的一篇blog：set uid;set gid; sticky bit区别](http://www.blogbus.com/wolfchina-logs/11948787.html)
