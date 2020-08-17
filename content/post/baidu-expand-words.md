---
title: "Golang 实现的 Baidu 拓词工具"
date: 2020-08-08T21:05:03+08:00
draft: false
description: "一个Golang 实现的拓词工具，使用不同方法拓词"
categories: ["SEO","GO","源码"]
tags: ["SEO","GO","拓词工具"]
summary: "一个Golang 实现的拓词工具，使用不同方法拓词"
slug: "baidu expand words"
---

# wordexpand 百度拓词工具
 
## 1. 安装
```shell script
  go get github.com/kevin-five/wordexpand@v0.0.1
```

## 2. 使用
总共两个工具
1. 通过百度搜索推荐词拓词，拓词个数还可以，但是有的词相关性不好，例子：
```

package main

import (
	"github.com/kevin-five/wordexpand/baidu/recommand"
	"log"
)

func main() {
	keywords, err := recommand.Expand("大象")
	if err != nil {
		log.Fatalf("when get baidu recommend words got: %+v\n", err)
	}
	for i, keyword := range keywords {
		log.Printf("expand word %d: %s\n", i+1, keyword)
	}
}

output:
2020/08/08 20:55:38 expand word 1: 大象电影网
2020/08/08 20:55:38 expand word 2: 大象电影
2020/08/08 20:55:38 expand word 3: 河南民生频道今天节目
....
2020/08/08 20:55:38 expand word 65: 大象电影简介
2020/08/08 20:55:38 expand word 66: 河南广播电视台卫星频道直播

```


2. 通过下拉词拓词，每次拓词10个及以下，但是相关性都不错，例子
```
package main

import (
	"github.com/kevin-five/wordexpand/baidu/suggest"
	"log"
)

func main() {
	keywords, err := suggest.PCExpand("大象")
	if err != nil {
		log.Fatalf("when get baidu suggest words got: %+v\n", err)
	}
	for i, keyword := range keywords {
		log.Printf("expand word %d: %s\n", i+1, keyword)
	}
}
output:
2020/08/08 20:58:26 expand word 1: 大象补发
2020/08/08 20:58:26 expand word 2: 大象英语
2020/08/08 20:58:26 expand word 3: 大象画室
2020/08/08 20:58:26 expand word 4: 大象图片
2020/08/08 20:58:26 expand word 5: 大象简笔画
2020/08/08 20:58:26 expand word 6: 大象小说软件下载
2020/08/08 20:58:26 expand word 7: 大象席地而坐
2020/08/08 20:58:26 expand word 8: 大象简笔画图片大全
2020/08/08 20:58:26 expand word 9: 大象叫声
2020/08/08 20:58:26 expand word 10: 大象的拼音

```
## 3. 版本
v0.0.1
这个版本随手写的，以后会继续，然后会再增加拓词渠道的
