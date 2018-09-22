+++
categories = ["技术"]
date = "2016-08-01T07:40:35-07:00"
draft = false
slug = ""
tags = ["hugo"]
title = "使用hugo搭建博客教程（二）"

+++

github pages部署个人博客

Github pages分为两种：一种是项目主页，每个项目都可以有一个；另一种是用户主页，一个用户只能有一个。


### 创建和部署

创建两个单独的git仓库repo：

	在Github上创建repo <your-project>-hugo，托管Hugo的输入文件。	
	创建repo <username>.github.io，用于托管public/文件夹。
	
	注意这里的repo名字一定要用自己的用户名，才会被当作是个人主页。
    
clone your-project

	$ git clone <<your-project>-hugo-url>
    eg: git clone https://github.com/<username>/<your-project>-hugo.git

进入your-project 目录

	$ cd <your-project>-hugo

删掉public目录（这个目录每次运行Hugo都会再次生成，不用担心）

	$ rm -rf public

把public/目录添加为submodule 与<username>.github.io同步

	$ git submodule add git@github.com:<username>/<username>.github.io.git public

    eg:git submodule  add https://github.com/<username>/<username>.github.io.git public

添加.gitignore文件，文件中写public/，在同步<your-project>-hugo时会忽略public文件夹

	vim  .gitignore
	public/

部署脚本，方便提交github和部署网站： deploy.sh 

	```
		#!/bin/bash
		source /etc/profile;
		echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"
		
		msg="rebuilding site `date`"
		
		echo $msg
		
		if [ $# -eq 1 ]
		  then msg="$1"
		fi
		
		# Push Hugo content 
		git add -A
		git commit -m "$msg"
		git push origin master
		
		
		# Build the project. 
		hugo  # if using a theme, replace by `hugo -t <yourtheme>`
		
		# Go To Public folder
		cd public
		# update 
		git pull
		# Add changes to git.
		git add -A
		
		# Commit changes.
		
		git commit -m "$msg"
		
		# Push source and build repos.
		git push origin master
		
		# Come Back
		cd ..
	```

部署系统：
	
	执行deploy.sh，浏览http://username.github.io/，查看效果。
	每次更新网站或者写了新文章，只需要运行./deploy.sh 发布即可。

Github pages域名绑定：    
    
自定义域名（个人域名）绑定，获取github的个人网站的ip点：

	$ ping <username>.github.io

去你的域名管理界面增加上两条A记录

	分别是www和@，记录指向http://<username>.github.io/ 的ip地址，也需要等一小会儿生效。

github-settings输入自定义域名：

	https://github.com/githoo/githoo.github.io/settings
	Custom domain : your-website.com
	
	或者在<username>.github.io repo的跟目录下添加CNAME文件，文件里写上你的域名，不用加http://的开头。

### 高级功能

#### 上传图片

上传图片方法：

	hugo site中上传图片路径：

	static/images/about/about.png

markdown中设置图片路径：

```
	markdown图片：![about](/images/about/about.png "test image")
```

#### 添加评论的方法：

	由于disqus访问不了，可以使用国内的类似的duoshuo

创建多说域名：

	http://duoshuo.com/create-site/ 

设置多说域名：

	http://yourduoshuoshortname.duoshuo.com/admin/

增加duoshuo模板文件：

	layouts/partials/duoshuo.html
	<!-- 多说评论框 start -->
	<div class="ds-thread" data-thread-key="{{ .URL }}" data-title="{{ .Title }}" data-url="{{ .Permalink }}"></div>
	<!-- 多说评论框 end -->

	<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
	<script type="text/javascript">
	    var duoshuoQuery = {short_name:"{{ .Site.Params.comments.duoshuoShortname }}"};

	    (function() {
	     var ds = document.createElement('script');
	     ds.type = 'text/javascript';ds.async = true;
	     ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
	     ds.charset = 'UTF-8';
	     (document.getElementsByTagName('head')[0]
	      || document.getElementsByTagName('body')[0]).appendChild(ds);
	     })();
	    </script>
	<!-- 多说公共JS代码 end -->

修改page模板文件：

	layouts/_default/single.html
	{{ partial "duoshuo.html" . }}


#### 统计访问量

注册statcounter账号：

	http://zh_cn.statcounter.com

增加statCounter模板文件：

	layouts/partials/statCounter.html
	<!-- Start of StatCounter Code for Default Guide -->
	<script type="text/javascript">
	var sc_project=11072857;
	var sc_invisible=0;
	var sc_security="98129f58";
	var scJsHost = (("https:" == document.location.protocol) ?
	"https://secure." : "http://www.");
	document.write("<sc"+"ript type='text/javascript' src='" +
	scJsHost+
	"statcounter.com/counter/counter.js'></"+"script>");
	</script>
	<noscript><div class="statcounter"><a title="website
	statistics" href="http://statcounter.com/"
	target="_blank"><img class="statcounter"
	src="//c.statcounter.com/11072857/0/98129f58/0/"
	alt="website statistics"></a></div></noscript>
	<!-- End of StatCounter Code for Default Guide -->

修改显示模板：

	layouts/partials/ footer.html
	<span> 
	              visits {{ partial "statCounter.html" . }}
	</span>


