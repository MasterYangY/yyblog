---
layout: post
title: 'Github Pages搭建个人博客过程'
date: 2018-03-13
author: YY
cover: '/assets/img/pages.jpg'
tags:  GithubPages jekyll
---

# 1.安装Ruby+jekyll

下载地址[https://rubyinstaller.org/downloads/archives/](https://rubyinstaller.org/downloads/archives/ )

考虑到版本问题，需要下载`rubyinstaller-2.3.3-x64.exe`和`DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe`这两个安装。

安装之后命令行输入
```
$ gem install jekyll
```

安装jekyll


# 2.通过模板创建网站

先用github创建新repository，然后用github desktop把这个库的master同步到本地文件夹中。

下载模板，本站套用的模板下载地址[jekyll-theme-h2o](http://jekyllthemes.org/themes/jekyll-theme-h2o/)

把模板解压到同步下来的本地文件夹中，在该文件夹打开CMD输入命令

```
jekyll serve --safe --watch
```

然后浏览器打开[http://localhost:4000](http://localhost:4000)

如果能看到模板网站说明部署成功。


# 3.修改模板

[https://github.com/kaeyleo/jekyll-theme-H2O](https://github.com/kaeyleo/jekyll-theme-H2O)有详细的模板修改方法。

# 4.发布文章
把.md格式的文件放到`_posts`文件夹里，具体格式参考模板范文。

