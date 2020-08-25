---
title: "基于文本及符号密度的网页正文提取方法 golang实现"
date: 2020-08-21T22:05:27+08:00
draft: false
description: "最近在做无监督的新词提取工作，然后发现自己找的语料有问题，主要的问题是抓的不是正文，所以尝试了一下提取正文。"
categories: ["SEO"]
tags: ["seo","语料处理","golang"]
summary: "基于文本及符号密度的网页正文提取方法 golang实现"
slug: "web page text extract"
---

# 网页正文提取工作

## 目的
最近在做无监督的新词提取工作，然后发现自己找的语料有问题，主要的问题是抓的不是正文，有很多诸如网站名，网站活动等的噪声，所以尝试了一下提取正文。于是找到了这篇论文 [基于文本及符号密度的网页正文提取方法 (知网地址) ](http://www.cnki.com.cn/Article/CJFDTotal-GWDZ201908029.htm)，并基于golang简单的实现了一个beta版本这里做一下记录。


一定不要直接用在生产环境上，只是我看了文章后手痒自己实现了一下，这个论文有些地方写的不是特别的严谨，但是大体方向是ok的，还有部分可以改进的地方我稍后再做做

## 代码
```
package article

import (
	"fmt"
	"github.com/PuerkitoBio/goquery"
	"math"
	"strings"
	"unicode"
)

type NodeInfo struct {
	T     int     // 结点字符串字数
	LT    int     // 结点带连接的字符串的字数
	TG    int     // 结点标签数
	LTG   int     // 结点带连接的标签数
	PNum  int     //
	TD    float64 //
	Score float64
	Sb    float64
	SbD   float64
}
type Article struct {
	Title       string
	Summary     string
	ContentText string
	ContentHTML string
}

func ExtractArticle(html string) (*Article, error) {
	infoMap, err := calculate(html)
	if err != nil {
		return nil, err
	}
	maxScore := 0.0
	var maxNode *goquery.Selection
	for node, info := range infoMap {
		if info.Score > maxScore {
			maxScore = info.Score
			maxNode = node
		}
	}
	if maxNode == nil {
		return nil, fmt.Errorf("extract article err, can't get content node")
	}
	a := Article{}
	a.ContentHTML, err = goquery.OuterHtml(maxNode)
	if err != nil {
		return nil, err
	}
	a.ContentText = maxNode.Text()
	a.Title, a.Summary, err = getArticleInfo(html)
	return &a, err
}

func getArticleInfo(html string) (title string, summary string, err error) {
	doc, err := goquery.NewDocumentFromReader(strings.NewReader(html))
	if err != nil {
		return "", "", err
	}
	title = doc.Find("H1").Text()
	if title == "" {
		title = doc.Find("title").Text()
	}
	doc.Find("meta").EachWithBreak(func(_ int, meta *goquery.Selection) bool {
		if strings.Contains(meta.AttrOr("name", ""), "description") || strings.Contains(meta.AttrOr("property", ""), "description") {
			summary = meta.AttrOr("content", "")
			return false
		}
		return true
	})
	return
}

func computeInfo(node *goquery.Selection, nodeMap map[*goquery.Selection]*NodeInfo) *NodeInfo {
	var nodeInfo = &NodeInfo{}
	if node.Children().Size() > 0 {
		node.Children().Each(func(_ int, child *goquery.Selection) {
			childNodeInfo := computeInfo(child, nodeMap)
			nodeInfo.T += childNodeInfo.T
			nodeInfo.LT += childNodeInfo.LT
			nodeInfo.TG += childNodeInfo.TG
			nodeInfo.LTG += childNodeInfo.LTG
			nodeInfo.PNum += childNodeInfo.PNum
			nodeInfo.Sb += childNodeInfo.Sb
		})
		nodeInfo.TG += 1
		nodeMap[node] = nodeInfo
	} else {
		if node.Is("a") {
			nodeInfo.LTG++
			nodeInfo.LT = len(node.Text())
		} else if node.Is("p") {
			nodeInfo.PNum += 1
		}

		nodeInfo.TG = 1
		nodeInfo.T = len(node.Text())
		if nodeInfo.T > 0 {
			for _, r := range node.Text() {
				if unicode.IsPunct(r) {
					nodeInfo.Sb += 1
				}
			}
		}

	}

	return nodeInfo
}

func calculate(html string) (map[*goquery.Selection]*NodeInfo, error) {
	nodeMap := make(map[*goquery.Selection]*NodeInfo)
	doc, err := removeScriptAndStyle(html)
	if err != nil {
		return nil, err
	}
	body := doc.Find("body")
	computeInfo(body, nodeMap)
	sum := 0.0

	for _, info := range nodeMap {
		td := float64(info.T-info.LT) / float64(info.TG-info.LTG+1)
		info.TD = td
		sum += td
		info.SbD = float64(info.T-info.LT) / (info.Sb + 1.0)
	}
	nodeInfoCount := float64(len(nodeMap))
	avg := sum / nodeInfoCount

	sdp := 0.0
	for _, info := range nodeMap {
		sdp += math.Pow(info.TD-avg, 2) / (nodeInfoCount)

	}
	sd := math.Sqrt(sdp)

	for _, info := range nodeMap {
		info.Score = math.Log(sd) * info.TD * math.Log10(float64(info.PNum+2)) * math.Log(info.SbD+1)
	}
	return nodeMap, nil
}

func removeScriptAndStyle(html string) (*goquery.Document, error) {
	doc, err := goquery.NewDocumentFromReader(strings.NewReader(html))
	if err != nil {
		return nil, err
	}
	doc.Find("script").Each(func(_ int, s *goquery.Selection) {
		scriptHtml, err := goquery.OuterHtml(s)
		if err != nil {
			return
		}
		html = strings.ReplaceAll(html, scriptHtml, "")
	})

	doc.Find("style").Each(func(_ int, s *goquery.Selection) {
		scriptHtml, err := goquery.OuterHtml(s)
		if err != nil {
			return
		}
		html = strings.ReplaceAll(html, scriptHtml, "")
	})
	return goquery.NewDocumentFromReader(strings.NewReader(html))

}

```

使用方法:

```
    html,err := http_util.GetWebConFromUrl("https://news.163.com/20/0821/04/FKHDIVVD0001899O.html")
	if err != nil {
		panic(err)
	}
	article,err := ExtractArticle(html)
	if err != nil {
		panic(err)
	}
	fmt.Printf("title: %s\n",article.Title)
	fmt.Printf("summary: %s\n", article.Summary)
	fmt.Printf("content: %s\n", article.ContentText)
	fmt.Printf("contentHtml: %s\n", article.ContentHTML)
```