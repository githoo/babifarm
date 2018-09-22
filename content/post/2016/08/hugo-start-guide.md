+++
categories = ["技术"]
date = "2016-08-01T06:06:46-07:00"
draft = false
slug = ""
tags = ["hugo"]
title = "使用hugo搭建博客教程（一）"

+++

## hugo 初体验 

### hugo安装：

1. 下载：
	https://github.com/spf13/hugo/releases/download/v0.16/hugo_0.16_linux-64bit.tgz
	
1. 安装：
	tar xzvf hugo_0.16_linux-64bit.tgz  -C hugo_0.16_linux

### hugo使用：

1. 建网站：

	$ hugo new site mysite
	$ cd mysite
	在该目录下你可以看到以下几个目录和 config.toml 文件 
	- ▸ archetypes/ 
	- ▸ content/
	- ▸ layouts/
	- ▸ static/
	-   config.toml

	config.toml 是网站的配置文件，包括 baseurl , title , copyright 等等网站参数。
	这几个文件夹的作用分别是：

	* archetypes：包括内容类型，在创建新内容时自动生成内容的配置
	* content：包括网站内容，全部使用markdown格式
	* layouts：包括了网站的模版，决定内容如何呈现
	* static：包括了css, js, fonts, media等，决定网站的外观


1. 创建新页面：

	$ hugo new about.md
	相关操作都在mysite目录下，否则会报错；
	进入 content/ 文件夹可以看到，此时多了一个markdown格式的文件 about.md ，打开文件可以看到时间和文件名等信息已经自动加到文件开头，包括创建时间，页面名，是否为草稿等。

1. 下载主题：

	$  git clone https://github.com/spf13/hyde.git

1. 运行Hugo:

	$ hugo server -t hyde --buildDrafts  --verbose

	-t 参数的意思是使用hyde主题渲染我们的页面，注意到 about.md 目前是作为草稿，即 draft 参数设置为 true ，运行Hugo时要加上 --buildDrafts 参数才会生成被标记为草稿的页面。The “--verbose” flag gives extra information that will be helpful when we build the template  

1. 在浏览器中输入如下地址，查看刚才创建的页面：

	http://localhost:1313

