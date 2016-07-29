layout: post
title: 利用Tesseract图片文字识别初探
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [tesseact,]
date: 2016-07-28 08:58:19
keywords: tesseract, 图片文字识别
---

![tesseract-to-pages](/images/blog/tesseract-to-pages.png)

一直以来都想尝试一下图片的中文识别，直到最近才有点空闲时间，主要目的是证实一下到底可不可行，正确率能否达到 95% 以上。自己从头写起十分费时间，因为图片处理很麻烦，所以我选用了`Tesseract OCR`。

所谓 `OCR`(Optical Character Recognition)是指对文本资料进行扫描，然后对图像文件进行分析处理，获取文字和版面信息的过程。`OCR`是图像识别领域中的一个子领域，该领域专注于对图片中的文字信息进行识别并转换成能被常规文本编辑器编辑的文本。

<!-- more -->
## Tesseract介绍

`Tesseract`(/'tesərækt/) 这个词的意思是"超立方体"，指的是几何学里的四维标准方体，又称"正八胞体"，是一款被广泛使用的开源 `OCR` 工具。

![tesseract](http://linusp.github.io/assets/img/tesseract.gif)

`Tesseract` 已经有 30 年历史，开始它是惠普实验室于1985年开始研发的一款专利软件，到1995年一件成为OCR业界内最准确的识别引擎之一。然而，HP不久便决定放弃OCR业务，`Tesseract`从此尘封。数年之后，HP意识到与其将Tesseract束之高阁，还不如贡献给开源，让其重焕新生。在 2005 年，`Tesseract`由美国内华达州信息技术研究所获得，并求助于`Google`对`Tesseract`进行改进、消除Bug、优化工作，并开源，其后一直由 `Google` 赞助进行后续的开发和维护。因为其免费与较好的效果，许多的个人开发者以及一些较小的团队在使用着 `Tesseract` ，诸如验证码识别、车牌号识别等应用中，不难见到 `Tesseract` 的身影。

现在`Tesseract`托管在`Github`上，大家有兴趣可以上`Github`上`Star`或`Frok`[该项目](https://github.com/tesseract-ocr/tesseract)。

## 安装

### Mac OSX

在`Mac`上安装`Tesseract`是一件非常简单的事情，我们还是使用`brew`来进行安装。


```bash
brew install tesseract
```


```bash
➜ tesseract --version
tesseract 3.04.01
 leptonica-1.73
  libjpeg 8d : libpng 1.6.23 : libtiff 4.0.6 : zlib 1.2.5
```

不过，如果你只是用上述命令来安装Tesseract的话，就会发现，只支持英文，因为它只默认安装了`eng`的语言包。如果我们需要识别其他的语言该如何来办呢？

安装指定的语言包：

```bash
brew intsall tesseract
cd /usr/local/Cellar/tesseract/{version}/share/tessdata
wget https://github.com/tesseract-ocr/tessdata/raw/master/chi_sim.traineddata
```

使用`brew`安装所有语言包：

```bash
brew install tesseract --all-languages
```

### 其他平台安装

更多Tesseract的安装可以查看这儿[Install Tesseract via pre-built binary package](https://github.com/tesseract-ocr/tesseract/wiki)或 [build it from source](https://github.com/tesseract-ocr/tesseract/wiki/Compiling)

## 命令行使用

这里只见到讲一下Tesseract识别图像的基本用法，关于训练和开发将来在另开新篇来专门讲述。

由于Tesseract只提供命令行工具，这里讲的用法对于Linux和Windows平台都适用。

首先可以通过`"--list-langs"`来查看哪些可用的“语言”，如果之前的`TESSDATA_PREFIX`环境变量没有设置错误，将看到这样的输出：

```bash
➜  ~tesseract --list-langs
List of available languages (107):
afr
amh
ara
asm
aze
aze_cyrl
bel
ben
bod
bos
bul
cat
ceb
ces
chi_sim
chi_tra
chr
cym
dan
.....
```

大家可以看到，我安装了107个语言包，其中,`eng`和`chi_sim`是`Tesseract`提供的英文和简体中文的语言文件。

另外，要说明的是，这里的 "语言文件" 的本质是包含了某种 "自然语言" 的文字的特征等辅助识别的一些资源，但像 `chi_sim` 这个中文简体里也包含了英文字母与阿拉伯数字的资源。而我们也可以为了特定的用途而去训练产生对应的资源，并且可以给这个资源自定义一个名字。

如果发现以上命令的输出为空，那应该去检查一下 `TESSDATA_PREFIX` 这个环境变量。在这个环境变量无误且 "语言文件" 存在的情况下，假设我们有一张名为 `paper.png` 的图片，则通过以下命令对图片进行识别，

```bash
tesseract paper.png paper -l chi_sim
```

1. 第一个参数是待识别的图像的文件名
1. 第二个参数用于指定输出，如果希望直接输出而不是保存到文件，那么就使用 `stdout`，否则这个参数将会作为保存结果的文件的前缀
1. -l `chi_sim` 这个应该很好理解，就是用来指定使用哪个 "语言文件"，如果是使用 英文(`eng`) ，这个参数可以不加，因为默认就是使用英文的 "语言文件" 来进行识别

> 以上命令如不出错，结果将会保存到 paper.txt 这个文本文件中。

此外 `Tesseract` 还提供非常丰富的可选参数来对识别过程进行调整，可用的参数及其默认值可以通过以下命令进行查看:

```bash
tesseract --print-parameters
```

参数的使用有两种:

* 使用 -c 选项来设定单项参数的值，比如:

```bash
tesseract paper.png paper -l chi_sim -c language_model_ngram_on=1
```

允许使用多个 -c 选项来设置多个参数的值。

* 将多项参数设置写入文件，然后在识别时使用该文件，比如:

```bash
tesseract paper.png paper -l chi_sim tess.conf
```

需要注意的是，如果使用配置文件，用作参数的配置文件名要放在最后面——这里也支持多个配置文件，但它们必须要在最后面。假如我有两个配置文件 tess_1.conf 和 tess_2.conf，那么这样是正确的:
```bash
tesseract paper.png paper -l chi_sim tess_1.conf tess_2.conf
```

而这样则是错误的:

```bash
tesseract paper.png paper tess_1.conf -l chi_sim tess_2.conf
```

至于 `Tesseract` 那些参数各有什么含义，官方没有提供任何文档来进行解释，这里有一个[链接](http://www.sk-spell.sk.cx/tesseract-ocr-parameters-in-302-version)提供了部分参数的用处说明，应该是阅读了 `Tesseract` 源代码后得到的结论。

## 另外

关于`Tesseract`的文档，可以查看[Tetesseract官方Blog](http://tesseract-ocr.github.io/index.html
)

