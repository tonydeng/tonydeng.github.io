layout: post
title: 批量转换文件编码
published: false
comments: true
sharing: true
footer: true
categories: [效率]
tags: [shell, iconv]
date: 2015-11-27 12:11:11
keywords: shell , iconv
---
# 需求

之前同事有一个项目给过来，由于他之前的开发环境是Windows的，文件编码都是GBK的，看起挺不爽的，不符合我们现在的规范。需要将里面的文件全部转换成UTF8的文件编码。

那我们应该怎么来做呢？

很简单，写一个shell，利用 `iconv`转换一下就好了，于是花了5分钟左右写了一个脚本来搞定这个事情。

<!-- more -->

# 脚本

```
#!/bin/bash

DIR=$1 # 转换编码文件目录
FT=$2  # 需要转换的文件类型（扩展名）
SE=$3  # 原始编码
DE=$4  # 目标编码

for file in `find $DIR -type f -name *.$FT`; do
	echo "conversion $file encoding $SE to $DE"
    iconv -f $SE -t $DE "$file" > "$file".tmp
    mv -f "$file".tmp "$file"
done
```

该脚本已经提交到[github](https://github.com/tonydeng/note/blob/1594ae267114effa910ff2511176d3dbf7968471/sh/batch_conversion_encoding.sh)上。

# 调用方式

```
➜  ~ ./batch_conversion_encoding.sh ~/sdk1 java GBK UTF8
```