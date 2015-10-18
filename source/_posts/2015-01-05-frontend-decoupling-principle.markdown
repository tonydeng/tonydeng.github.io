---
layout: post
title: "前端解耦原则"
date: 2015-01-05 18:29:52 +0800
published: true
comments: true
categories: [程序设计]
tags: [front-end, engineered]
keywords: 前端 解耦
description: 当你能够做到修改一个组件而不需要更改其他组件时，就做到了松耦合，松耦合对于代码可维护性来说是至关重要的。
---

![解耦](/images/blog/decoupling.png)

## 松耦合

当你能够做到修改一个组件而不需要更改其他组件时，就做到了松耦合，松耦合对于代码可维护性来说是至关重要的。

### 原则

* 不要使用 css 表达式。（这种方式应该已经废弃了）
* javascript 和 css 之间只通过 className 进行通信。
* 不要使用 html 的 on 属性，如：onclick。应该使用 id 保持 javascript 和 html 的沟通。
* 使用模板。

## 将 **javascript** 从 **css** 中抽离

在 ie8 和更早版本的浏览器中有一个特性，即 **css 表达式** 。如下：

###  不好的写法

```css
.box {
    width: expression(document.body.offsetWidth + "px");
}
```

不过，现在ie9 已经不再支持 **css 表达式** 了。
而且，这种写法自2013年以来，我再没有见过。

## 将 **css** 从 **javascript** 中抽离

### 不好的写法
```javascript
element.style.color = "red";
element.style.left = "10"px;
element.style.visibility = "visible";
```

### 好的写法

在 css 中定义样式 ， 在 javascript 中 通过 className 通信
```css
.reveal {
    color: red;
    left: 10px;
    visibility: visible;
}
```

javascript

```javascript
element.className += "reveal";    // 原生写法
$(element).addClass("reveal");    // 如果是 jQuery
```

## 将 **javascript** 从 **html** 中抽离

### 不好的写法

```
<button onclick="doSomething()" id="action">click me</button>
```

我们要避免使用 onclick 等 on 属性来绑定一个事件处理程序。
应该使用 id 保持 javascript 和 html 的沟通。

### 好的写法, 如使用 jQuery
```
$("#action").on("click",doSomething);
```

## 将 **html** 从 **javascript** 中抽离

注释中包含模板文本

```
<ul id="mylist"><!--<li id="item%s"><a href="%s">%s</a></li>
    <li><a href="1">First item</a></li>
    <li><a href="2">Second item</a></li>
    <li><a href="3">Third item</a></li>
</ul>
```

### 注释也是 **DOM** 的一部分


function addItem(url, text) {
    var mylist = document.getElementById("mylist"),
        templateText = mylist.firstChild.nodeValue;
        result = sprintf(template, url, text);
    div.innerHTML = result;
    mylist.insertAdjacentHTML("beforeend", result);
}
addItem("4", "Four item");


### 使用一个带有自定义 **type** 属性的 **script** 元素。

浏览器会默认地将 **script** 元素中的内容识别为 **JavaScript** 代码， 但你可以通过给 **type** 赋值为浏览器不识别的类型，来告诉浏览器这不是一段 javascript 脚本。

```
<script type="text/x-my-template" id="list-item">
    <li><a href="%s">%s</a></li>
</script>
```

你可以通过 **script** 标签的 **text** 属性来提取模板文本。
这样 **addItem()** 函数就会变成这样。

```
function addItem(url, text) {
    var mylist = document.getElementById("mylist"),
        script = document.getElementById("list-item"),
        templateText = script.test,
        result = sprintf(template, url, text),
        div = document.createElement("div");
    div.innerHTML = result.replace(/^\s*/, "");
    list.appendChild(div.firstChild);
}
// 用法
addItem("/item/4", "Four item");
```
