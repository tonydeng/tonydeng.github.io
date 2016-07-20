layout: post
title: 看看程序猿是如何来下载和处理视频
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [video, ffmpeg,youtube-dl]
date: 2016-07-19 16:14:58
keywords: 看看程序猿是如何来下载和处理视频 视频 video ffmpeg 合并视频 youtube-dl 下载视频
---

![FFMpeg and Youtube-dl](/images/blog/ffmpeg-youtube-dl.png)

我们经常在各大视频网站上看到一些很不错的视频，希望能够下载收藏起来，但是无法在网站上找到下载的按钮或者入口，很是郁闷。

那么来看看程序猿是如何来下载和处理视频的。

<!-- more -->

## 演示

今天我看到了一个英国拍的工程师吐槽的视频，准备下载拿来做分享的素材。但是你如果在优酷上遍寻不到可以直接下载的链接，只有使用优酷的客户端才能给下载。

对于程序猿来说，这个事情实在是太让人不爽了，那我们该怎么办？

### YouTube-DL

那就该视频下载神器`youtube-dl`入场。别看名字和youtube相关，其实它支持大部分的视频网站。比如，我们现在要下载视频的优酷。


我们来试试`youtube-dl`的使用。

```bash
➜ youtube-dl http://v.youku.com/v_show/id_XNzE1NTk3Mzky.html

[youku] XNzE1NTk3Mzky: Downloading JSON metadata
[download] Downloading playlist: 工程师的痛只有工程师能懂_高清
[youku] playlist 工程师的痛只有工程师能懂_高清: Collected 2 video ids (downloading 2 of them)
[download] Downloading video 1 of 2
[download] Destination: 工程师的痛只有工程师能懂_高清-XNzE1NTk3Mzky_part1.flv
[download] 100% of 7.82MiB in 00:07
[download] Downloading video 2 of 2
[download] Destination: 工程师的痛只有工程师能懂_高清-XNzE1NTk3Mzky_part2.flv
[download] 100% of 7.73MiB in 00:10
[download] Finished downloading playlist: 工程师的痛只有工程师能懂_高清
```

### FFMpeg

很简单就下载了这个视频，不过新的问题就来了，我下载下来的视频有两个，怎么能合并成一个文件呢？

嗯，应该引入我们另外一个神器`FFMpeg`,这是一个在视频领域里面家喻户晓的类库+工具，我之前做的视频相关的应用就是利用了`FFMpeg`。有兴趣的同学可以去看看我开源的项目[FMJ](https://github.com/tonydeng/fmj)。

那继续看看，我们怎么利用`FFMpeg`合并视频的。

```bash
ffmpeg -i "工程师的痛只有工程师能懂_高清-XNzE1NTk3Mzky_part1.flv" -c copy -bsf:v h264_mp4toannexb -f mpegts 1.ts
ffmpeg -i "工程师的痛只有工程师能懂_高清-XNzE1NTk3Mzky_part2.flv" -c copy -bsf:v h264_mp4toannexb -f mpegts 2.ts
ffmpeg -i "concat:1.ts|2.ts" -c copy -bsf:a aac_adtstoasc "工程师的痛只有工程师能懂.mp4"
```

这样我们就解决了下载和视频处理的需求，看起来是不是和使用鼠标点点点来完成这些事情不一样呢？

其实，我这儿还只是粗浅的演示，更多的方式可以，去[Youtube-DL官网](http://youtube-dl.org/)、[youtube-dl的开源项目地址](https://github.com/rg3/youtube-dl/),[FFMpeg官网](http://ffmpeg.org/)看看更详细的介绍和使用说明。

## 参考

另外，也可以看看其他人总结的一些更详细的经验和组合的自动化脚本工具。

比如：

1. [youtube-dl高级使用方法，混合参数下载](http://www.5yun.org/7636.html)
2. [优酷付费vip视频下载方法](http://www.5yun.org/8224.html)
3. [使用youtube-dl下载视频的开源脚本](https://github.com/kashu/ydl.sh)
4. [自动批量合并视频文件](https://github.com/kashu/merge.videos)

附上这个视频吧,有兴趣的人可以看看。


<iframe height=498 width=510 src="http://player.youku.com/embed/XNzE1NTk3Mzky" frameborder=0 allowfullscreen></iframe>
