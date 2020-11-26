layout: post
title: DFX中的可测试性是什么？
published: true
comments: true
sharing: true
footer: true
categories: [架构设计]
tags: [架构设计, DFX, 可测试性]
date: 2020-11-26 11:38:58
keywords: DFX中的可测试性是什么？, 架构设计, DFX, 可测试性
---

## 为什么引入可测试性设计？

在讨论可测试性之前，我们不妨先来思考一个问题：

当我们来盖一栋楼，如果不能保证钢筋、水泥、砖土质量合格，却想要盖出合格的大楼来，这个可能吗？

然而，很多团队的软件开发就是这么做的。很荒唐吧！

![DFX类型](/images/blog/DFX/DFX-Type.png)

在非功能性需求中，执行质量是很多程序员的心头所爱，一般也不会被忽略。但**演化质量的地位却很低，常常为人忽略，尤其是其中的“可测试性”**。

可测试性为什么如此重要？因为我们做设计，其实就是把**一个软件拆分成一个一个的小模块**。

如果不尽可能地保证每个小模块的正确性，而只是从最外围的系统角度去验证系统的正确性，这将会是一个非常困难的过程。

## 可测试性的定义

软件的可测试性是指在**一定时间和成本前提下**，进行**测试设计、测试执行**以此来**发现软件的问题**，以及**发现故障并隔离、定位其故障**的能力特点。

采用[James Bach](https://en.wikipedia.org/wiki/James_Marcus_Bach)的定义，**可测试性就是一个计算机程序能够被测试的容易程度**。

 我设计了一个公式来用于计算 **软件可测试性 = (测试方法覆盖度 * 测试资源保障度) / 测试目标**

> 维基百科中是这样定义[软件可测试性(Software testability)](https://zh.wikipedia.org/wiki/%E8%BB%9F%E9%AB%94%E5%8F%AF%E6%B8%AC%E8%A9%A6%E6%80%A7)：**一个软件工件（软件系统、组件、需求文档或设计文档等）在一给定的测试环境下，可支持测试的程度**。

一般来说，可测试性好的软件必然是一个**强内聚**、**弱耦合**、**接口明确**、**意图清晰**的软件。

## 可测试性的分析

那可测试性到底包括什么呢？我们从不同的维度来看可测试性的特征。

### 角色和职责

> 不同的岗位在可测试性中的角色

![Actors](/images/blog/DFX/testability/actors.png)

### 可测试性的各个视角

> 从不同的视角来看可测试性包含哪些特征

![可测试性的各个视角](/images/blog/DFX/testability/Usecase.png)

### 可观察

> 你所看见的就是你所测试的

![可观察用例](/images/blog/DFX/testability/can-be-viewed-usecase.png)

### 可操作

> 运行的越好，被测试的效率越高

![可操作用例](/images/blog/DFX/testability/operability-usecase.png)

### 可控制

> 对软件控制的越好，测试越能够被自动执行与优化

![可控制用例](/images/blog/DFX/testability/can-be-controlled-usecase.png)

#### 可理解性

> 得到的信息越多，进行的测试越准确、精细

![可理解性用例](/images/blog/DFX/testability/can-understand-usecase.png)

#### 简单性

> 需要测试的内容越少，测试的速度越快

![简单性用例](/images/blog/DFX/testability/simplicity-usecase.png)

#### 稳定性

> 改变越少，对测试的破坏越小

![稳定性用例](/images/blog/DFX/testability/stability-usecase.png)

#### 可分解

> 通过控制测试范围，能够更好分解问题，执行测试

![可分解性用例](/images/blog/DFX/testability/decomposability-usecase.png)