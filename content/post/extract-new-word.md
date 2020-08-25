---
title: "基于相互信息和信息熵获取新词算法 golang实现版本"
date: 2020-08-25T21:31:45+08:00
draft: false
description: "我的需求是无监督取词，并能自动识别新词，行业特殊关键词。 然后有了这篇基于相互信息和信息熵获取新词算法 golang实现版本"
categories: ["SEO"]
tags: ["SEO","关键词","分词"]
summary: "基于相互信息和信息熵获取新词算法 golang实现版本"
slug: "extract new word"
---
>使用的资源：  
[matrix67 博文地址](http://www.matrix67.com/blog/archives/5044)  
[数学之美-吴军 淘宝](https://s.click.taobao.com/6JpNDxu)  
[数学之美-吴军 豆瓣](https://book.douban.com/subject/26163454/)  
>参考代码：  
[python 实现](https://github.com/c19/ChineseSlicer.git)

我的需求是无监督取词，并能自动识别新词，行业特殊关键词。
先是看了吴军大神的 《数学之美》  第6章 信息的度量和作用 | 第4章 谈谈分词 这两个章节有所启发。
之后发现 matrix67 大神的文章，谈了具体的实现和应用。
最后参照了上面的代码实现了golang版本。

废话不多说，show code！*(还未过多做测试，生产环境请自行编写单元测试，还有许多要调优的地方，但是基本可用了)*

**核心代码**
```
package extractor

import (
	"math"
	"strings"
)

type Builder struct {
}

type Word struct {
	Word     string
	LeftNei  []string
	RightNei []string
	Freq     float64
	Poly     float64 //凝合度
	Flex     float64
	Point    float64
	WordLen  int
}

type Words []*Word

func (w Words) Len() int {
	return len(w)

}

func (w Words) Less(i, j int) bool {
	return w[i].Point < w[j].Point
}

func (w Words) Swap(i, j int) {
	w[i], w[j] = w[j], w[i]
}

func GenWords(corpus string, maxLen int) map[string]*Word {
	words := make(map[string]*Word)
	corpusParts := strings.Split(corpus, "")
	totalLen := len(corpusParts)
	for j := 1; j < maxLen; j++ {
		for i := range corpusParts {
			if i >= totalLen-j+1 {
				break
			}
			w := strings.Join(corpusParts[i:i+j], "")
			var word *Word
			if wd, ok := words[w]; ok {
				wd.Freq += 1
				word = wd
			} else {
				word = &Word{
					Word:     w,
					LeftNei:  []string{},
					RightNei: []string{},
					WordLen:  j,
					Freq:     1,
				}
				words[w] = word
			}

			if j > 1 {
				left := ""
				right := ""
				if i-1 > 0 {
					left = corpusParts[i-1]
					word.LeftNei = append(word.LeftNei, left)
				}
				if i+j+j <= totalLen {
					right = corpusParts[i+j]
					word.RightNei = append(word.RightNei, right)
				}

			}
		}
	}
	for _, word := range words {
		word.Freq = word.Freq / float64(totalLen*word.WordLen)
	}
	return words
}

func cutter(str string) [][]string {
	var r [][]string
	for i := range str {
		if i == 0 {
			continue
		}
		r = append(r, []string{str[0:i], str[i:]})
	}
	return r
}

// 得分综合
func PointCount(words map[string]*Word) map[string]*Word {
	for _, word := range words {
		if word.WordLen > 1 {
			word.Poly = polyCount(word, words)
			word.Flex = flexCount(word)
			word.Point = pointCountCalculate(word)
		}
	}
	return words

}

func pointCountCalculate(word *Word) float64 {
	if word.WordLen <= 1 {
		return 0
	}
	return word.Freq * word.Poly * word.Flex
}

// 自由度
func flexCount(word *Word) float64 {
	return math.Min(flexCountCalculate(word.LeftNei), flexCountCalculate(word.RightNei))
}

func flexCountCalculate(neighbors []string) float64 {
	l := float64(len(neighbors))
	if l == 0 {
		return 1
	}
	entropy := 0.0
	for _, c := range counter(neighbors) {
		ps := float64(c) / l
		entropy += ps * math.Log(ps)
	}
	return -entropy
}

func counter(nei []string) map[string]int {
	cm := make(map[string]int)
	for _, s := range nei {
		if _, ok := cm[s]; ok {
			cm[s] += 1
		} else {
			cm[s] = 1
		}

	}
	return cm
}

// 凝合度
func polyCount(word *Word, words map[string]*Word) float64 {
	if word.WordLen == 1 {
		return 1
	}
	m := 0.0
	for _, cutPiece := range cutter(word.Word) {
		c := words[cutPiece[0]].Freq * words[cutPiece[1]].Freq
		if c > m {
			m = c
		}
	}
	if m <= 0 {
		return 1
	}
	return word.Freq / m
}

```

**代码使用 西游记关键词抽取**

```
raw, err := ioutil.ReadFile("data/xyj.txt")
if err != nil {
    panic(err)
}
data := string(raw)
// data = clear.PureCorpus(data, "")
wordMap := cpcode.GenWords(data, 5)
wordMap = cpcode.PointCount(wordMap)
words := cpcode.Words{}
for _, word := range wordMap {
    words = append(words, word)
}
sort.Sort(words)
for _, word := range words {
    fmt.Printf("k: %s freq:%.2f poly:%.4f flex:%.4f point:%.4f \n", word.Word, word.Freq*1000, word.Poly, word.Flex, word.Point)
}

# output 节选

# k: 怎么 freq:0.66 poly:103.5259 flex:4.1662 point:0.2830 
# k: 氤氲 freq:0.01 poly:32763.8333 flex:1.4751 point:0.2868 
# k: 弼马温 freq:0.03 poly:3951.0119 flex:2.6460 point:0.2895 
# k: 悟空 freq:0.43 poly:197.8959 flex:3.4222 point:0.2940 
# k: 喧哗 freq:0.02 poly:5670.6635 flex:2.9125 point:0.2940 
# k: 蘑菇 freq:0.01 poly:28668.3542 flex:1.7479 point:0.2974 
# k: 仔细 freq:0.11 poly:918.6121 flex:3.0735 point:0.3016 
# k: 模样 freq:0.13 poly:772.5411 flex:2.9420 point:0.3025 
# k: 鸳鸯 freq:0.01 poly:29487.4500 flex:1.7479 point:0.3059 
# k: 毂辘 freq:0.02 poly:11072.3626 flex:1.7326 point:0.3090 
# k: 包袱 freq:0.05 poly:1866.2943 flex:3.1337 point:0.3124 
# k: 芭蕉 freq:0.05 poly:4396.8781 flex:1.3822 point:0.3195 
# k: 乒乓 freq:0.02 poly:8424.9857 flex:2.2468 point:0.3210 
# k: 褊衫 freq:0.01 poly:10531.2321 flex:2.2696 point:0.3242 
# k: 琵琶 freq:0.01 poly:22682.6538 flex:1.3269 point:0.3317 
# k: 虼蚤 freq:0.00 poly:58974.9000 flex:1.3322 point:0.3330 
# k: 揭谛 freq:0.07 poly:1787.1182 flex:2.6288 point:0.3346 
# k: 惫懒 freq:0.03 poly:4834.0082 flex:2.7656 point:0.3400 
# k: 娉婷 freq:0.00 poly:73718.6250 flex:1.3863 point:0.3466 
# k: 三藏 freq:1.17 poly:75.7083 flex:3.9596 point:0.3512 
# k: 荆棘 freq:0.02 poly:6300.7372 flex:2.7334 point:0.3650 
# k: 魏征 freq:0.04 poly:3295.1153 flex:3.1322 point:0.3763 
# k: 蜻蜓 freq:0.01 poly:16381.9167 flex:1.8065 point:0.3764 
# k: 珊瑚 freq:0.01 poly:36859.3125 flex:1.7479 point:0.3823 
# k: 蜈蚣 freq:0.01 poly:49145.7500 flex:1.5607 point:0.3902 
# k: 葡萄 freq:0.01 poly:29487.4500 flex:1.7351 point:0.3904 
# k: 琥珀 freq:0.00 poly:58974.9000 flex:1.6094 point:0.4024 
# k: 玳瑁 freq:0.00 poly:58974.9000 flex:1.6094 point:0.4024 
# k: 踌躇 freq:0.00 poly:58974.9000 flex:1.6094 point:0.4024 
# k: 霹雳 freq:0.01 poly:26806.7727 flex:1.6663 point:0.4166 
# k: 魍魉 freq:0.01 poly:32763.8333 flex:1.6770 point:0.4192 
# k: 琉璃 freq:0.02 poly:8672.7794 flex:2.6651 point:0.4507 
# k: 狻猊 freq:0.01 poly:29487.4500 flex:2.0432 point:0.4597 
# k: 麒麟 freq:0.01 poly:17345.5588 flex:1.9928 point:0.4689 
# k: 玄奘 freq:0.08 poly:1629.2409 flex:3.8042 point:0.4729 
# k: 师父 freq:1.38 poly:96.8987 flex:3.5676 point:0.4786 
# k: 萧瑀 freq:0.02 poly:12145.8604 flex:2.6593 point:0.4929 
# k: 猢狲 freq:0.05 poly:4745.1069 flex:2.2110 point:0.4981 
# k: 朦胧 freq:0.02 poly:11672.1156 flex:2.6526 point:0.4987 
# k: 芙蓉 freq:0.01 poly:32763.8333 flex:2.0432 point:0.5108 
# k: 缥缈 freq:0.02 poly:15519.7105 flex:2.1832 point:0.5171 
# k: 哪吒 freq:0.06 poly:2843.9475 flex:2.9890 point:0.5261 
# k: 囫囵 freq:0.01 poly:19658.3000 flex:2.1186 point:0.5297 
# k: 骷髅 freq:0.01 poly:17345.5588 flex:2.2740 point:0.5351 
# k: 玲珑 freq:0.02 poly:13339.5607 flex:2.5209 point:0.5417 
# k: 玛瑙 freq:0.01 poly:26806.7727 flex:2.2719 point:0.5680 
# k: 崎岖 freq:0.01 poly:21062.4643 flex:2.4583 point:0.5707 
# k: 和尚 freq:0.67 poly:272.2865 flex:3.2464 point:0.5906 
# k: 乾坤 freq:0.04 poly:4246.4646 flex:3.5097 point:0.6318 
# k: 行者 freq:3.68 poly:45.5297 flex:3.8908 point:0.6514 
# k: 葫芦 freq:0.08 poly:2730.3194 flex:3.3470 point:0.6973 
# k: 峥嵘 freq:0.02 poly:14041.6429 flex:2.8216 point:0.7054 
# k: 八戒 freq:1.53 poly:125.9621 flex:3.9206 point:0.7570 
# k: 袈裟 freq:0.13 poly:1890.2212 flex:3.4589 point:0.8592 
# k: 吩咐 freq:0.11 poly:2233.5012 flex:3.7849 point:0.9174 
# k: 菩萨 freq:0.66 poly:367.6388 flex:4.6197 point:1.1274 
```

