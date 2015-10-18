---
layout: post
title: "使用HTML5的应用缓存"
date: 2015-01-13 21:56:27 +0800
published: true
comments: true
categories: [性能]
tags: [html5, front-end, performance]
keywords:
description:
---
![manifest](/images/blog/manifest.png)

# 简介

 [HTML5](https://html5.org/) 提供一种 **应用缓存** 机制，使得基于web的应用程序可以离线运行。开发者可以使用 Application Cache (AppCache) 接口设定浏览器缓存的数据并使得数据离线有效。 在处于离线状态时，即使用户点击刷新按钮，应用也能正常加载与工作。

使用应用缓存可以得到以下益处：

* 离线浏览: 用户可以在离线状态下浏览网站内容。
* 更快的速度: 因为数据被存储在本地，所以速度会更快.
* 减轻服务器的负载: 浏览器只会下载在服务器上发生改变的资源。

# 应用缓存如何工作
## 开启应用缓存
若想为应用开启应用缓存，你需要在应用页面中的 <html> 元素上增加 manifest 特性，请看下面的例子：

```html
<html manifest="example.appcache">
  ...
</html>
```

manifest 特性与 **缓存清单(cache manifest)** 文件关联，这个文件包含了浏览器**需要为你的应用缓存的资源(文件)列表**。

你应当在每一个意图缓存的页面上添加 **manifest** 特性。浏览器不会缓存不带有**manifest** 特性的页面，除非这个页面已经被写在清单文件内的列表里了。你没有必要添加所有你意图缓存的页面的清单文件，浏览器会将用户访问过的并带有 manifest 特性的所有页面添加进应用缓存中。

## 加载文档

使用了应用缓存机制以后加载文档的顺序是这样的：

* 如果应用缓存存在，浏览器直接从缓存中加载文档与相关资源，不会访问网络。这会提升文档加载速度。
* 浏览器检查清单文件列出的资源是否在服务器上被修改。
* 如果清单文件被更新了, 浏览器会下载新的清单文件和相关的资源。 这都是在后台执行的，基本不会影响到webapp的性能。

下面详细描述了加载文档与更新应用缓存的流程：

1. 当浏览器访问一个包含 manifest 特性的文档时，如果应用缓存不存在，浏览器会加载文档，然后获取所有在清单文件中列出的文件，生成应用缓存的第一个版本。
1. 对该文档的后续访问会使浏览器直接从应用缓存(而不是服务器)中加载文档与其他在清单文件中列出的资源。此外，浏览器还会向 window.applicationCache 对象发送一个 checking 事件，在遵循合适的 HTTP 缓存规则前提下，获取清单文件。
1. 如果当前缓存的清单副本是最新的，浏览器将向 applicationCache 对象发送一个 noupdate 事件，到此，更新过程结束。注意，如果你在服务器修改了任何缓存资源，同时也应该修改清单文件，这样浏览器才能知道它需要重新获取资源。
1. 如果清单文件已经改变，文件中列出的所有文件—也包括通过调用 applicationCache.add() 方法添加到缓存中的那些文件—会被获取并放到一个临时缓存中，遵循适当的 HTTP 缓存规则。对于每个加入到临时缓存中的文件，浏览器会向 applicationCache 对象发送一个 progress 事件。如果出现任何错误，浏览器会发送一个 error 事件，并暂停更新。
1. 一旦所有文件都获取成功，它们会自动移送到真正的离线缓存中，并向  applicationCache 对象发送一个 cached 事件。鉴于文档早已经被从缓存加载到浏览器中，所以更新后的文档不会重新渲染，直到页面重新加载(可以手动或通过程序).

# 存储位置与清除离线缓存

在 Chrome 中，你可以在设置中选择 「清除浏览器数据...」 或访问 [chrome://appcache-internals/](chrome://appcache-internals/) 来清除缓存。

Safari 在设置中有一个类似的"清空缓存" 选项，但是需要重启浏览器后才能生效。

在 Firefox 中，离线缓存数据与 Firefox 配置文件是分开存储的—紧挨着硬盘缓存：

* Windows Vista/7: C:\Users\<username>\AppData\Local\Mozilla\Firefox\Profiles\<salt>.<profile name>\OfflineCache
* Mac/Linux: /Users/<username>/Library/Caches/Firefox/Profiles/<salt>.<profile name>/OfflineCache

应用缓存可以变成废弃的。如果从服务器上移除一个应用的清单文件，浏览器将会清除所有清单中列出的应用缓存，并向 applicationCache 对象发送一个「obsolete」事件。这将使得应用缓存的状态变为 OBSOLETE。


# 缓存清单文件

## 引用一个缓存清单文件

web 应用中的 manifest 特性可以指定为缓存清单文件的相对路径或一个绝对 URL(绝对 URL 必须与应用同源)。缓存清单文件可以使用任意扩展名，但传输它的 MIME 类型必须为 text/cache-manifest。

Apache配置对manifest支持，在根目录或应用的同级目录下的**.htaccess**中加入下面的配置

```
AddType text/cache-manifest .appcache
```

Nginx配置对manifest支持，在**conf/mime.types**中加入下面的配置

```
text/cache-manifest                  appcache manifest;
```

## 缓存清单文件中的记录

缓存清单文件是一个纯文本文件，它列出了所有浏览器应该缓存起来的资源，以便能够离线访问。资源使用 URI 来标识。在缓存清单文件中列出的所有记录必须拥有相同的协议、主机名与端口号。

# 示例 1：一个简单的缓存清单文件

下面是一个简单的缓存清单文件，example.appcache，适用于一个虚拟的网站 www.example.com。

```
CACHE MANIFEST
# v1 - 2011-08-13
# This is a comment.
http://www.example.com/index.html
http://www.example.com/header.png
http://www.example.com/blah/blah
```

一个缓存清单文件可以包含三段内容 (CACHE， NETWORK， 和 FALLBACK， 下面详细讨论)。 在上面的例子中，没有段落标题，因此所有数据行都认为是属于显式 (CACHE) 段落，这意味着浏览器应该在应用缓存中缓存所有列出的资源。资源可以使用绝对或者相对 URL 来指定(例如 index.html)。

上面例子中的注释 「v1」很有必要存在。只有当清单文件发生变化时，浏览器才会去更新应用缓存。如果你要更改缓存资源(比如说，你使用了一张新的 header.png 图片)，你必须同时修改清单文件中的内容，以便让浏览器知道它们需要更新缓存。你可以对清单文件做任何改动，但大家都认同的最佳实践则是修正版本号。

> 重要：不要在清单文件中指定清单文件本身，否则将无法让浏览器得知清单文件有新版本出现。

## 缓存清单文件中的段落： CACHE， NETWORK，与 FALLBACK

清单文件可以分为三段： CACHE， NETWORK，与 FALLBACK.

**CACHE:**
    这是缓存文件中记录所属的默认段落。在 CACHE: 段落标题后(或直接跟在 CACHE MANIFEST 行后)列出的文件会在它们第一次下载完毕后缓存起来。

**NETWORK:**
    在 NETWORK: 段落标题下列出的文件是需要与服务器连接的白名单资源。所有类似资源的请求都会绕过缓存，即使用户处于离线状态。可以使用通配符。

**FALLBACK:**
    FALLBACK: 段指定了一个后备页面，当资源无法访问时，浏览器会使用该页面。该段落的每条记录都列出两个 URI—第一个表示资源，第二个表示后备页面。两个 URI 都必须使用相对路径并且与清单文件同源。可以使用通配符。

CACHE， NETWORK， 和 FALLBACK 段落可以以任意顺序出现在缓存清单文件中，并且每个段落可以在同一清单文件中出现多次。


# 示例 2： 一个复杂且完整的缓存清单文件

下面是一个更加完整的缓存清单文件，适用于一个虚拟的网站 www.example.com：

```
CACHE MANIFEST
# v1 2011-08-14
# This is another comment
index.html
cache.html
style.css
image1.png

# Use from network if available
NETWORK:
network.html

# Fallback content
FALLBACK:
/ fallback.html
```

该例子使用了 NETWORK 与 FALLBACK 段落，指明了 network.html 页面必须始终从网络获取，fallback.html 页面应该作为后备资源来提供(例如，当无法与服务器建立连接时)。

## 缓存清单文件的结构

缓存清单文件必须以 **text/cache-manifest** MIME 类型来传输。所有通过 MIME 类型传输的文件必须符合本节中定义的适用于应用缓存清单的语法。

缓存清单是 UTF-8 格式的文本文件，有可能包含一个 BOM 字符。新行可能使用换行符(U+000A)，回车(U+000D)，或回车加换行符来表示。

缓存清单文件的第一行必须包含字符串 CACHE MANIFEST (两个单词间使用一个 U+0020 空白)，紧接着是零或多个空白或制表符。本行的其他文本会被忽略。

缓存清单文件的余下内容必须包含零或多个下面的行：

**空行**
    你可以使用包含零或多个空白与制表符的空行。

**注释**
    注释包括零或多个制表符或空白字符，紧接着是一个 # 字符，再然后是零或多个注释文本字符。注释只能在所在行起作用，不能追加到其他行上。这意味着你无法使用片段标识符。

**段落标题**
    段落标题指定了缓存文件即将操作的段落。有三个可选的标题：

        CACHE: 	    切换到缓存清单的显式段落(默认段落)。
        NETWORK: 	切换到缓存清单的在线白名单段落。
        FALLBACK: 	切换到缓存清单的后备资源段落。

缓存清单可以在段落内任意切换(每个段落标题可以使用多次)，而且段落允许为空。

# 一个应用缓存中的资源

一个应用缓存至少会包含一个资源，由 URI 指定。所有资源都属于下列类别之一：

**主记录**
    这些资源被加入缓存的原因是：用户浏览的一个上下文中包含一个文档，该文档用 manifest 特性明确指明了它属于该缓存。

**显式记录**
    这些是在应用缓存清单文件中显式列出的资源。

**网络记录**
    这些是在应用缓存清单文件中作为网络记录列出的资源。

**后备记录**
    这些是在应用缓存清单文件中作为后备记录列出的资源。

## 主记录
任意在 <html> 元素上包含一个 manifest 特性的 HTML 文件都可以是主记录。例如，我们拥有 HTML 文件 http://www.example.com/entry.html ，它看起来是这样的：

```html
<html manifest="example.appcache">
  <h1>Application Cache Example</h1>
</html>
```

如果 entry.html 没有在 example.appcache 缓存清单文件中列出来，那么访问 entry.html 页面会使得 entry.html 作为一条主记录加入到应用缓存中。

## 显式记录

显式记录就是在缓存清单文件的 CACHE 段落显式列出的资源。

## 网络记录

缓存清单文件的 NETWORK 段落指定了 web 应用需要在线访问的资源。一个应用缓存中的网络记录本质上来说是一个「在线白名单」—在 NETWORK 段落指定的 URI 会从服务器而不是缓存加载。这使得浏览器的安全模型通过限制用户让其只访问经过验证的资源来避免潜在的安全漏洞。

举例来说，你可以使用网络记录来从服务器而不是缓存中加载并执行脚本或其他代码：

```
CACHE MANIFEST
NETWORK:
/api
```

上面列出的缓存清单段落能够保证对 http://www.example.com/api/ 子目录中资源的请求始终通过网络加载，而不会去访问缓存。

> 注意： 简单的从清单文件中过滤主记录(在 html 元素中拥有 manifest 特性的文件)并不会产生同样的结果，因为主记录会被添加到—后续访问的获取也会从—应用缓存中。

## 后备记录

当尝试请求资源失败时会使用后备记录。例如，缓存清单文件 http://www.example.com/example.appcache 包含如下内容：

```
CACHE MANIFEST
FALLBACK:
example/bar/ example.html
```

任何访问 http://www.example.com/example/bar/ 或它的任意子目录及内容都会使浏览器发出请求，去尝试加载请求的资源。如果尝试失败(可能是由于网络连接失败或服务器问题)，浏览器将会加载 example.html。


# 缓存状态

每个应用缓存都有一个状态，标示着缓存的当前状况。共享同一清单 URI 的缓存拥有相同的缓存状态，可能是其中之一：

**UNCACHED(未缓存)**
    一个特殊的值，用于表明一个应用缓存对象还没有完全初始化。

**IDLE(空闲)**
    应用缓存此时未处于更新过程中。

**CHECKING(检查)**
    清单已经获取完毕并检查更新。

**DOWNLOADING(下载中)**
    下载资源并准备加入到缓存中，这是由于清单变化引起的。

**UPDATEREADY(更新就绪)**
    一个新版本的应用缓存可以使用。有一个对应的事件 updateready，当下载完毕一个更新，并且还未使用 swapCache() 方法激活更新时，该事件触发，而不会是 cached 事件。

**OBSOLETE(废弃)**
    应用缓存现在被废弃。

# 测试缓存清单的更新

你可以使用 JavaScript 来写程序检测应用是否拥有一个可以更新的缓存清单文件。因为缓存清单文件可能会在脚本添加事件前完成更新，所以脚本应该始终检测 window.applicationCache.status。

```javascript
function onUpdateReady() {
  alert('found new version!');
}
window.applicationCache.addEventListener('updateready', onUpdateReady);
if(window.applicationCache.status === window.applicationCache.UPDATEREADY) {
  onUpdateReady();
}
```

 若要手动测试一个新的清单文件，你可以使用 ```window.applicationCache.update()``` 。

# 陷阱

* 永远不要使用传统 GET 参数(例如 ```other-cached-page.html?parameterName=value```) 来访问缓存文件。这会使浏览器绕过缓存，直接从网络获取。若想链接一个参数需要在 JavaScript 中解析的资源，你可以将参数放到链接的 hash 部分，例如 ```other-cached-page.html#whatever?parameterName=value``` 。

* 当应用被缓存后，仅仅更新在 web 页面中使用的资源(文件)还不足以更新被缓存的文件。你需要在浏览器获取和使用更新的文件前，去更新缓存清单文件本身。你可以使用 ```window.applicationCache.swapCache()``` 以编程的方式完成上述目的，虽然这无法影响到已经加载完毕的资源。为了保证资源从应用缓存的最新版本中加载，最理想的办法就是刷新页面。

* 通过在 web 服务器上设置 ```expires header``` 来使 ```*.appcache``` 文件立即过期是个好主意。这避免了将清单文件缓存的风险。例如，在 Apache 中，你可以指定下面的配置项：
```
    ExpiresByType text/cache-manifest "access plus 0 seconds"
```

## 另见

* [HTML5Rocks - A Beginner's Guide to Using the Application Cache](http://www.html5rocks.com/en/tutorials/appcache/beginner/)
* [appcachefacts.info](http://appcachefacts.info/) - detailed information on AppCache idiosyncrasies
* [offline web applications](http://hacks.mozilla.org/2010/01/offline-web-applications/) at hacks.mozilla.org - showcases an offline app demo and explains how it works.
* [HTML 5 working draft: Offline web applications](http://www.whatwg.org/specs/web-apps/current-work/multipage/offline.html#offline)
* [HTML5 Cache Manifest: An Off-label Usage](http://developer.teradata.com/blog/js186040/2011/11/html5-cache-manifest-an-off-label-usage)
* [nsIApplicationCache](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIApplicationCache)
* [nsIApplicationCacheNamespace](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIApplicationCacheNamespace)
* [nsIApplicationCacheContainer](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIApplicationCacheContainer)
* [nsIApplicationCacheChannel](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIApplicationCacheChannel)
* [nsIApplicationCacheService](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIApplicationCacheService)
* [nsIDOMOfflineResourceList](https://developer.mozilla.org/zh-CN/docs/XPCOM_Interface_Reference/nsIDOMOfflineResourceList)
* [Get ready for Firefox 3.0 - A Web developer's guide to the many new features in this popular browser, especially the offline application features](http://www.ibm.com/developerworks/web/library/wa-ffox3/) (IBM developerWorks)
