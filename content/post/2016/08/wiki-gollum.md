+++
categories = ["技术"]
date = "2016-08-02T06:06:46-07:00"
draft = false
slug = ""
tags = ["wiki","tool"]
title = "wiki-gollum 安装说明"

+++

 
1.安装RVM https://rvm.io/

    ssh admin@192.168.1.1
    curl -L get.rvm.io | bash -s stable 
    source ~/.bashrc
    source ~/.bash_profile
2.修改 RVM 的 Ruby 安装源到淘宝服务器(可以不改)

    sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db
3.安装系统依赖包

    CentOS
    yum install libicu libicu-devel zlib zlib-devel git
4.安装python 2.7+ (一般系统都有自带)

5.安装ruby 1.9.3+

    rvm install 1.9.3
    rvm list //可以查看目前已经安装的包
    rvm use 1.9.3 --default //如果有多个版本时设置指定版为默认
    rvm remove 1.9.0 //卸载一个不用的版本
6.安装gollum

    gem install gollum

7.配置git仓库

    ssh admin@192.168.1.2
    cd /export/App
    mkdir wiki.git
    cd wiki.git
    git init --bare

7.启动并配置wiki

    ssh admin@192.168.1.1
    cd /export/APp
    git clone admin@172.16.162.55:/export/App/wiki.git
    cd wiki
    gollum --base-path wiki --port 4567 --mathjax

8.配置域名访问

    ssh admin@192.168.1.3
    nginx.conf:
    server{
       location /wiki{
           proxy_pass  http://192.168.1.1:4567;
       }
    }
    sbin/nginx -s reload

9.访问wiki

    http://si.jd.com/wiki

10.备份wiki

    ssh admin@192.168.1.1
    crontab -e:
    */20 * * * * sh /export/App/wiki_bak_shell/bak.sh >> /export/App/wiki_bak_shell/log.txt 2>&1 &
    