---
title: "golang 中计算 Unicode|中文|utf-8 字符串的长度"
date: 2020-08-17T11:53:37+08:00
draft: false
description: "golang 中计算 Unicode|中文|utf-8 字符串的长度"
categories: ["技术","go"]
tags: ["golang"]
summary: "golang 中计算 Unicode|中文|utf-8 字符串的长度"
slug: "golang count unicode string length"
---

# golang 中计算 Unicode|中文|utf-8 字符串的长度

```
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    count := utf8.RuneCountInString("测试字符串 abc 123 测试")
    fmt.Print(count) // 16
}
```
