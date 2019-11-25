---
layout: post
title: 'Hostinger+Wordpress自建博客记录'
date: 2018-03-28
author: YY
cover: 'https://yyblog.win/assets/img/20180328wordpress.png'
tags:  自学记录
---
<h1>1.域名申请</h1>
<a href="https://wanwang.aliyun.com/" target="_blank">阿里云（万网）</a>直接买，.win域名十年的价格是50RMB，第一次买要实名制备案，需要一天左右。

拿到域名之后不要解析。
<h1>2.空间申请</h1>
在<a href="https://www.hostinger.com.hk/" target="_blank">https://www.hostinger.com.hk/</a>上点无限空间然后选哪个免费套餐。

结账结束之后进客户中心，在控制面板建立新主机，选免费的那个。

期间填已经注册好的域名，一律下一步结束。

打开主机主页会看到
<h5>您的域名未指向我们的NS服务器，所以FTP、文件管理、邮箱及其他服务不能正常工作。您可以在"账号 -&gt; 信息"选项中找到我们的NS服务器信息。请注意DNS修改后将在24小时后生效。</h5>
<h1>3.添加解析</h1>
打开 【控制面板 - 账号 - 信息】

看到一堆NS服务器，打开这个页面不动，回到阿里云。

阿里云域名管理里点新买的域名，点【DNS修改/创建】，点【修改域名DNS】，把ns1.hostinger.com.hk等四个NS服务器粘贴进去，确定。

然后回到hostinger控制面板，反复刷新，直到不显示【您的域名未指向我们的NS服务器】为止。
<h1>4.安装Wordpress</h1>
打开 【控制面板 - 网站 - 一键安装助手】，找到Wordpress，点进去安装，注意此处如果想直接用域名当博客地址的话，网址那个空就空着不填。管理员的帐号不可更改，一定要用个好的。

安装完之后一分钟左右刷新就能看到【已安装版本】里有一个Wordpress，点击信息可以获取博客地址和管理员地址等。
<h1>5.调试Wordpress使用</h1>
看个人的喜好，可以看网站

<a href="https://www.wpdaxue.com/" target="_blank">WordPress大学</a>

&nbsp;

<!--more-->

1.Wordpress安装好后就可以在阿里云里把DNS改回来了，如果用CDN加速的话就先别改，设置CDN的时候还需要再改。本站CDN加速用的是<a href="http://su.baidu.com/" target="_blank">百度云加速</a>，设置方法见百度云加速的帮助。

2.侧边栏里放文本框，里面可以用html代码，实现一些功能。但是背景音乐不好实现，因为每次点开文章都会重新加载所有部分，背景音乐也会重新开始。所以最下面的背景音乐本来应该是整个博客的背景音乐，无奈不能用了。

3.做事情不要想当然，本地看代码的效果放到服务器上可能完全就不一样了。

4.有功夫是该再学学HTML了。

<embed src="//music.163.com/style/swf/widget.swf?sid=720565761&type=0&auto=0&width=310&height=430" width="330" height="450"  allowNetworking="all">