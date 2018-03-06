layout: post
title: Linux中setuid、setgid及sticky bit的区别
published: true
comments: true
sharing: true
footer: true
categories: [安全]
tags: [linux, setuid, setgid, sticky bit]
date: 2007-04-06 17:46:56
keywords: linux setuid setgid sticky bit
---


![Linux File System Access Control](/images/blog/linux-access-control.png)

`Linux`中除了常见的读（`r`）、写（`w`）、执行（`x`）权限以外，还有3个特殊的权限，分别是`setuid`、`setgid`和`stick bit`。

这些特殊的权限，在被执行时都会有特殊的用处。如果不了解，很容易被黑客利用这些权限的特性来获得系统更高级的权限，甚至对系统造成无法挽回的伤害。

<!-- more -->
## 了解

- `setuid`: 在执行时具有文件所有者的权限.
- `setgid`: 设置目录. 一个目录被标上`setgid`位,此目录下创建的文件继承该目录的属性.
- `sticky bit`: 该位可以理解为防删除位. 设置`sticky bit`位后，就算用户对目录具有写权限，但也只能添加文件而不能删除文件。

## 系统已有的特殊权限相关文件

### `setuid`

```bash
$ ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 27832 6月  10 2006 /usr/bin/passwd
```

```bash
$ ls -l /usr/bin/chsh
-rws--x--x 1 root root 23872 8月   4 2006 /usr/bin/chsh
```

### `setgid`

```
$ ls -l /usr/bin/write
-rwxr-sr-x 1 root tty 19536 8月   4 2006 /usr/bin/write
```

### `sticky bit`

```bash
$ ls -l -d /tmp
drwxrwxrwt. 9 root root 198 3月   6 03:44 /tmp
```

## 如何设置：

操作这些标志与操作文件权限的命令是一样的, 都是使用`chmod`。

有两种方法对这些标志来操作

1) `chmod u+s temp` -- 为`temp`文件加上`setuid`标志. (`setuid` 只对文件有效，`u`=用户)
`chmod g+s tempdir` -- 为`tempdir`目录加上`setgid`标志 (`setgid` 只对目录有效，`g`=组名)
`chmod o+t temp` -- 为`temp`文件加上`sticky`标志 (`sticky`只对文件有效)

2) 采用八进制方式. 这一组八进制数字三位的意义如下,
`abc`
`a` - `setuid`位, 如果该位为`1`, 则表示设置`setuid `
`b` - `setgid`位, 如果该位为`1`, 则表示设置`setgid `
`c` - `sticky`位, 如果该位为`1`, 则表示设置`sticky`

设置后, 可以用 `ls -l` 来查看. 如果本来在该位上有`x`, 则这些特殊标志显示为小写字母 (`s, s, t`). 否则, 显示为大写字母 `(S, S, T`)
如:

`rwsrw-r--` 表示有`setuid`标志 (`rwxrw-r--:rwsrw-r--`)
`rwxrwsrw-` 表示有`setgid`标志 (`rwxrwxrw-:rwxrwsrw-`)
`rwxrw-rwt` 表示有`sticky`标志 (`rwxrw-rwx:rwxrw-rwt`)

## 扩展阅读

[CORE TECHNOLOGY: ACCESS CONTROL](https://www.linuxvoice.com/core-technology-access-control/)
[SADS: SetUid and SetGUID and sticky bit (Linux File Permissions)](https://www.youtube.com/watch?v=vKcb8M6mczA)
