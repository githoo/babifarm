
+++
categories = ["技术"]
date = "2016-08-13T13:06:46-07:00"
draft = false
slug = ""
tags = ["python"]
title = "利用Python远程拉取文件"

+++

  
利用Python自带的包可以建立简单的web服务器。

cd到准备做服务器根目录的路径下，输入命令：python -m Web服务器模块 [端口号，默认8000]


	$ python -m SimpleHTTPServer 8080

然后就可以在浏览器中输入:

	http://localhost:端口号/路径 来访问服务器资源。 

例如：

	http://localhost:8080/index.htm（当然index.htm文件得自己创建） 

实例：

	源头：192.168.1.1
	$ cd /home/admin/
	$ tar czvf test_deploy.tar.gz test_deploy
	$ python -m SimpleHTTPServer 8088

	目的：192.168.1.2
	$ cd /home/admin/;\rm -rvf test_deploy.tar.gz;wget http://192.168.1.1:8088/test_deploy.tar.gz;ll test_deploy.tar.gz|wc -l;tar -xzvf test_deploy.tar.gz;cd test_deploy;sh exe.sh;tail -f /export/servers/openresty/nginx/logs/error.log


