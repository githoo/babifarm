+++
categories = ["技术"]
date = "2016-08-02T09:06:46-07:00"
draft = false
slug = ""
tags = ["thrift"]
title = "thrift 教程"

+++

  
## thrift 安装  
thrift 版本：0.9.0
### thrift 依赖环境  
```xml
 A relatively POSIX-compliant *NIX system 
 Cygwin or MinGW can be used on Windows 
 g++ 3.3.5+ 
 boost 1.33.1+ (1.34.0 for building all tests) 
 Runtime libraries for lex and yacc might be needed for the compiler.  
```
### 安装步骤   
#### 1、安装依赖  
```xml 
$  yum install automake libtool flex bison pkgconfig gcc-c++ boost-devel libevent-devel zlib-devel python-devel ruby-devel 
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
c6-media | 4.0 kB 00:00
Setting up Install Process
Package automake-1.11.1-4.el6.noarch already installed and latest version
Package 1:pkgconfig-0.23-9.1.el6.x86_64 already installed and latest version
Package gcc-c++-4.4.7-3.el6.x86_64 already installed and latest version
Package zlib-devel-1.2.3-29.el6.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package bison.x86_64 0:2.4.1-5.el6 will be installed
---> Package boost-devel.x86_64 0:1.41.0-11.el6_1.2 will be installed
--> Processing Dependency: boost = 1.41.0-11.el6_1.2 for package: boost-devel-1.41.0-11.el6_1.2.x86_64
--> Processing Dependency: libboost_wserialization.so.5()(64bit) for package: boost-devel-1.41.0-11.el6_1.2.x86_64
--> Processing Dependency: libboost_wserialization-mt.so.5()(64bit) for package: boost-devel-1.41.0-11.el6_1.2.x86_64
....................................
```
#### 2、安装thrift   
  
 ```xml 
  $ tar -zxvf thrift-0.9.0.tar.gz   
  $ cd thrift-0.9.0   
  $ ./configure
  thrift 0.9.0

  Building C++ Library ......... : no
  Building C (GLib) Library .... : no
  Building Java Library ........ : no
  Building C# Library .......... : no 
  ......

  Python Library:
  Using Python .............. : /usr/bin/python
 ```
 继续执行以下步骤：
 ```xml 
   $ make (这步比较，耐心...)  
   $ make install    
 ```

检查是否安装成功：

```xml 
  $ thrift -version    
  Thrift version 0.9.0 （显示这个说明安装成功）  
``` 

### 生成Java 类文件
  
#### 1、定义thrift 文件  

```xml
$ vim Hello.thrift   
mespace java service.demo
 service Hello{
  string helloString(1:string para)
  i32 helloInt(1:i32 para)
  bool helloBoolean(1:bool para)
  void helloVoid()
  string helloNull()
 }
```

#### 2、生成Java 类文件  

```xml
 $ thrift -r -gen java ./Hello.thrift  // -r对其中include的文件也生成服务代码 -gen是生成服务代码的语言  

生成的Java 类文件：
 $ gen-java/service/demo/Hello.java
```
###  maven 插件配置
maven 项目的结构：  
```xml
 src
  |--main
  |--java
  |--resource
  |--thrift
       |--client.thrift(thrift文件)  
```  

插件配置  
```xml
 <plugin>
                <groupId>org.apache.thrift.tools</groupId>
                <artifactId>maven-thrift-plugin</artifactId>
                <version>0.1.12-SNAPSHOT</version>
                <configuration>
                    <thriftExecutable>/usr/local/bin/thrift</thriftExecutable>
                </configuration>
                <executions>
                    <execution>
                        <id>thrift-sources</id>
                        <phase>generate-sources</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```

###  官方文档
http://thrift.apache.org/tutorial/   
http://thrift.apache.org/tutorial/java
