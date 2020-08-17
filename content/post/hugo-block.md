---
title: "hugo 文档 — 基础模板和块(block)"
date: 2020-08-09T19:50:22+08:00
draft: false
description: "hugo 基础模板和块(block) 的使用"
categories: ["hugo"]
tags: ["hugo中文文档"]
summary: "hugo 基础模板和块(block) 的使用"
slug: "hugo block"
---


[官方文档](https://gohugo.io/templates/base/)

对 `基础模板`和`块（block）`的运用可以让你定义的主模板的框架


## 1. 定义基础模板

通过 `_default/baseof.html` 

还可以通过  `*baseof.html`  来覆盖基础模板。

```
# layouts/_default/baseof.html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>{{ block "title" . }}
      <!-- Blocks may include default content. -->
      {{ .Site.Title }}
    {{ end }}</title>
  </head>
  <body>
    <!-- Code that all your templates share, like a header -->
    {{ block "main" . }}
      <!-- The part of the page that begins to differ between templates -->
    {{ end }}
    {{ block "footer" . }}
    <!-- More shared code, perhaps a footer but that can be overridden if need be in  -->
    {{ end }}
  </body>
</html>
```

## 2. 覆盖基础模板

可以使用默认列表模板覆盖基础模板， 默认模板会继承所有基础模板的内容，然后通过实现自己的 `main` 块来定制内容

```
# layouts/_default/list.html
{{ define "main" }}
  <h1>Posts</h1>
  {{ range .Pages }}
    <article>
      <h2>{{ .Title }}</h2>
      {{ .Content }}
    </article>
  {{ end }}
{{ end }}
```

`define "main"` 的内容会替换基础模板中的块（block） `main` ，其它部分还是按照 `baseof.html` 的实现来渲染。

下面这个例子说明怎么同时覆盖 `main` 和 `title` 块

```
# layouts/_default/single.html
{{ define "title" }}
  <!-- This will override the default value set in baseof.html; i.e., "{{.Site.Title}}" in the original example-->
  {{ .Title }} &ndash; {{ .Site.Title }}
{{ end }}
{{ define "main" }}
  <h1>{{ .Title }}</h1>
  {{ .Content }}
{{ end }}
```