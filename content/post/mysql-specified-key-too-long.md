---
title: "使用 utf8mb4 编码时对字符串建立唯一索引报 mysql specified key too long 错误"
date: 2020-09-02T13:15:57+08:00
draft: false
description: "使用 utf8mb4 编码时对字符串建立唯一索引报 Specified key was too long; max key length is 767 bytes 错误"
categories: ["技术"]
tags: ["mysql"]
summary: "使用 utf8mb4 编码时对字符串建立唯一索引报 Specified key was too long; max key length is 767 bytes 错误"
slug: "mysql specified key too long"
---


## 问题描述
使用 utf8mb4 编码时建立唯一索引报 Specified key was too long; max key length is 767 bytes 错误

## 解决办法
1. 将 varchar(255) 改为 varchar(64)。
2. 启用 innodb_large_prefix, 限制就变为3072。

原因是utf8mb4 每个字符字节是 4。 用255超过了 767 这个长度的限制。 

[参考](https://blog.csdn.net/chenjianhuideyueding/article/details/88426021)