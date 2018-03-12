---
layout: post
title: 'Github Pages搭建个人博客测试'
date: 2018-03-11
author: YY
cover: 'http://i4.bvimg.com/636001/27251516694818f6.jpg'
tags:  GithubPages jekyll Markdown
---

#Markdown语法测试
####标题测试
分级标题
==
二级标题
-
[TOC]

####代码
> hello world!
> hello world!
> hello world!
####代码嵌套
>1
>>2
>>>3
####行内标记
代码块`hello world`代码块结束

```
<div>   
    <div></div>
    <div></div>
    <div></div>
</div>
```

{% highlight ruby %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}


	javascript
	var num = 0;
	for (var i = 0; i < 5; i++) {
	    num+=i;
	}
	console.log(num);
   
####超链接
[超链接1](http://www.baidu.com "百度一下")  
 
[超链接2][2]
[2]: http://www.baidu.com/   "百度二下"

####图片
图片1

![](https://www.baidu.com/img/baidu_jgylogo3.gif '图片1描述')

图片2

![name][01]
[01]: https://www.baidu.com/img/baidu_jgylogo3.gif  '图片2描述'

####图片带链接
[![](https://www.baidu.com/img/baidu_jgylogo3.gif '百度')](http://www.baidu.com)      // 内链式

[![](https://www.baidu.com/img/baidu_jgylogo3.gif '百度')][5]                    // 引用式
[5]: http://www.baidu.com


####插入视频

<iframe height=498 width=510 src='http://player.youku.com/embed/XMjgzNzM0NTYxNg==' frameborder=0 'allowfullscreen'></iframe>


[![](https://www.baidu.com/img/baidu_jgylogo3.gif)](http://v.youku.com/v_show/id_XMjgzNzM0NTYxNg==.html?spm=a2htv.20009910.contentHolderUnit2.A&from=y1.3-tv-grid-1007-9910.86804.1-2#paction)


####序表
1. one
2. two
3. three
#
* one
* two
* three
####序表嵌套
1. one
    1. one-1
    2. two-2
2. two 
    * two-1
    * two-2

#
* one
		
		var a = 10;     
####选项
这是文字……

- [x] 选项一
- [ ] 选项二  
- [ ]  [选项3]


####公式

$$ x \href{why-equal.html}{=} y^2 + 1 $$
$ x = {-b \pm \sqrt{b^2-4ac} \over 2a}. $

####分隔符
***
---
* * *

####注脚
Markdown[^1]
[^1]: Markdown是一种纯文本标记语言        // 在文章最后面显示脚注

####锚点
[公式标题锚点](#1)

### [需要跳转的目录] {#1}    // 方括号后保持空格

####解释型定义
Markdown 
:   轻量级文本标记语言，可以转换成html，pdf等格式  //  开头一个`:` + `Tab` 或 四个空格

代码块定义
:   代码块定义……

        var a = 10;         // 保持空一行与 递进缩进


####邮箱
<xxx@outlook.com>

####流程图
```flow                     // 流程
st=>start: 开始|past:> http://www.baidu.com // 开始
e=>end: 结束              // 结束
c1=>condition: 条件1:>http://www.baidu.com[_parent]   // 判断条件
c2=>condition: 条件2      // 判断条件
c3=>condition: 条件3      // 判断条件
io=>inputoutput: 输出     // 输出
//----------------以上为定义参数-------------------------

//----------------以下为连接参数-------------------------
// 开始->判断条件1为no->判断条件2为no->判断条件3为no->输出->结束
st->c1(yes,right)->c2(yes,right)->c3(yes,right)->io->e
c1(no)->e                   // 条件1不满足->结束
c2(no)->e                   // 条件2不满足->结束
c3(no)->e                   // 条件3不满足->结束
```

