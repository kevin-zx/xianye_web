---
title: "HUGO 在文章和模板中如何使用生产 URL — HUGO url 管理"
date: 2020-08-07T18:43:28+08:00
draft: false
description: "hugo url 管理使用 ref 和 relref 短代码来完成， ref 生成绝对链接， relref 生成相对链接"
categories: ["hugo"]
tags: ["hugo中文文档"]
summary: "hugo 中文文档， 关于如何在文章和模板中管理生产 url"
slug: "hugo url manage"
---

# HUGO 在文章中如何使用 URL — HUGO url管理

[英文官方文档](https://gohugo.io/content-management/cross-references/)

## 简单一句话：

使用 ref 和 relref 短码来完成这个操作
**（使用的时候去除{{和<符号中间的空格）**：

```java
// ref 是生成绝对链接
{{ < ref "document.md" > }}
{{ < ref "#anchor" > }}
{{ < ref "document.md#anchor" > }}
{{ < ref "/blog/my-post" > }}
{{ < ref "/blog/my-post.md" > }}

//relref 是生成相对链接
{{ < relref "document.md" > }}
{{ < relref "#anchor" > }}
{{ < relref "document.md#anchor" > }}

```

后面跟上路径和文件名，可以忽略路径和文件后缀。HUGO也会从当前路径开始寻找符合条件的文件。*但是有个问题是如果出现不能确定一致性的情况，HUGO就会报错。*

例子

```bash
# 在 markdown 文件中使用
[链接]({{ < ref "document.md#anchor" > }})

# 指定不同的语言
{{ < relref path="document.md" lang="ja" > }}

#指定不同的输出
{{ < relref path="document.md" outputFormat="rss" > }}

#锚文本
{{ < relref "#anchors" > }} => #anchors:9decaf7
{{ < relref "/blog/post.md#who" > }} => /blog/post/#who:badcafe

```

### 锚文本相关：

HUGO自动给 header 生成锚文本，所以只需要将 # 后面的锚文本换成 header 文字即可