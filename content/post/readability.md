---
title: "readability web 网页正文抽取工具介绍 golang实现版本"
date: 2020-08-24T21:48:48+08:00
draft: false
description: "最近想通过爬虫准备些语料，但是网页中有 header footer 等部分文字存在，常常会造成语料出现问题，难以清洗。于是产生一个需求，只获取网页正文，而不获取其它部分"
categories: ["SEO"]
tags: ["语料处理","golang","seo"]
summary: "readability web 网页正文抽取工具介绍 golang实现版本"
slug: "readability"
---

最近想通过爬虫准备些语料，但是网页中有 header footer 等部分文字存在，常常会造成语料出现问题，难以清洗。

于是产生一个需求，只获取网页正文，而不获取其它部分。

并且不通过特定规则来实现。然后找到了 golang 版本的 readability 试用后发现非常好用，这里做记录。

优点：好用，api简单易懂

缺点：不能保留格式，特殊需求不能满足

[go-readability github](https://github.com/go-shiori/go-readability)

```
#安装
go get -u -v github.com/go-shiori/go-readability
```

```
package main

import (
	"fmt"
	"log"
	"net/http"

	readability "github.com/go-shiori/go-readability"
)

var (
	urls = []string{
		// Both will be parse-able now
		"https://www.nytimes.com/2019/02/20/climate/climate-national-security-threat.html",
		// But this one will not have any content
		"https://www.nytimes.com/",
	}
)

func main() {
	for _, url := range urls {
		resp, err := http.Get(url)
		if err != nil {
			log.Fatalf("failed to download %s: %v\n", url, err)
		}
		defer resp.Body.Close()

		article, err := readability.FromReader(resp.Body, url)
		if err != nil {
			log.Fatalf("failed to parse %s: %v\n", url, err)
		}

		fmt.Printf("URL     : %s\n", url)
		fmt.Printf("Title   : %s\n", article.Title)
		fmt.Printf("Author  : %s\n", article.Byline)
		fmt.Printf("Length  : %d\n", article.Length)
		fmt.Printf("Excerpt : %s\n", article.Excerpt)
		fmt.Printf("SiteName: %s\n", article.SiteName)
		fmt.Printf("Image   : %s\n", article.Image)
		fmt.Printf("Favicon : %s\n", article.Favicon)
		fmt.Println()
	}
}
```