+++
categories = ["技术"]
date = "2018-12-23T10:06:46-07:00"
draft = false
slug = ""
tags = ["core"]
title = "neo4j remote access config and deploy"
+++





# neo4j远端访问配置及部署



## 1、neo4j的安装及配置

```
下载：neo4j-community-3.4.10-unix.tar.gz
wget https://neo4j.com/artifact.php?name=neo4j-community-3.4.10-unix.tar.gz

neo4j配置neo4j.conf:
cat /export/soft/neo4j-community-3.4.10/conf/neo4j.conf |grep '\='|grep -v '#'
 
dbms.directories.import=import
dbms.security.auth_enabled=false
dbms.connectors.default_listen_address=0.0.0.0
dbms.connector.bolt.enabled=true
dbms.connector.bolt.tls_level=OPTIONAL
dbms.connector.bolt.listen_address=:7687
dbms.connector.http.enabled=true
dbms.connector.http.listen_address=:7474
dbms.connector.https.enabled=true
dbms.connector.https.listen_address=:7473
dbms.tx_log.rotation.retention_policy=1 days
dbms.jvm.additional=-XX:+UseG1GC
dbms.jvm.additional=-XX:-OmitStackTraceInFastThrow
dbms.jvm.additional=-XX:+AlwaysPreTouch
dbms.jvm.additional=-XX:+UnlockExperimentalVMOptions
dbms.jvm.additional=-XX:+TrustFinalNonStaticFields
dbms.jvm.additional=-XX:+DisableExplicitGC
dbms.jvm.additional=-Djdk.tls.ephemeralDHKeySize=2048
dbms.jvm.additional=-Djdk.tls.rejectClientInitiatedRenegotiation=true
dbms.windows_service_name=neo4j
dbms.jvm.additional=-Dunsupported.dbms.udc.source=tarball

neo4启动：
/export/soft/neo4j-community-3.4.10/bin/neo4j start
```





## 2、nginx的安装及配置

### nginx安装

```
#nginx安装
wget 'http://nginx.org/download/nginx-1.9.2.tar.gz'
tar xzvf nginx-1.9.2.tar.gz
cd nginx-1.9.2
./configure --prefix=/export/soft/nginx --with-stream
make
make install
/export/soft/nginx/sbin/nginx -c /export/soft/nginx/conf/domains/pg.test.com
```

### nginx配置

```
#nginx配置 pg.test.com:

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    #include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    server {
        listen       80;
        server_name  pg.test.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;
    

        location /pg/ {
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header Host $http_host;
                    proxy_redirect off;
                    proxy_buffering off;
                    proxy_pass http://127.0.0.1:7474/browser/;
         }

        location / {
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                    proxy_set_header Host $http_host;
                    proxy_redirect off;
                    proxy_buffering off;
                    proxy_pass http://127.0.0.1:7687/;
         }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

}




```

### nginx启动

```
/export/soft/nginx/sbin/nginx -c /export/soft/nginx/conf/domains/pg.test.com
```



## 3、neo4j 访问配置

```
#配置hosts
10.10.188.88 pg.test.com

#neo4j-browser: 
http://pg.test.com/neo4j/

Connect to Neo4j:
Connect URL: bolt://pg.test.com:80
Username:neo4j
Password:neo4j
```

## 4、neo4j访问语句

```
- 查询产品词相似关系：
MATCH p=(n:PwNode{name:'口罩'})-[r:similar]->() RETURN p LIMIT 25

```



