layout: post
title: Hexo Mermaid流程图/序列图插件
published: true
comments: true
sharing: true
footer: true
categories: [效率]
tags: [hexo, mermaid,]
date: 2017-04-14 16:31:48
keywords: hexo mermaid plugin hexo-tag-mermaid
---

![Mermaid](https://knsv.github.io/mermaid/images/header.png)
Mermaid是

[Mermaid Live Editor](https://knsv.github.io/mermaid/live_editor/)

{% mermaid %}
  graph TB
           subgraph one
           a1-->a2
           end
           subgraph two
           b1-->b2
           end
           subgraph three
           c1-->c2
           end
           c1-->a2
{% endmermaid %}
