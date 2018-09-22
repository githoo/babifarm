+++
categories = ["技术"]
date = "2016-08-02T09:16:46-07:00"
draft = false
slug = ""
tags = ["protobuf"]
title = "protobuf 教程"

+++


   
 protobuf  版本 2.5.0 ，因为Hadoop hbase 都是这个版本 为了兼容 ，请用是用这个版本 

## maven插件生成

### protobuf 文件 
```xml
package p13.search; 


message BookCid3Inst
{
    required string cid3 = 1; //三级分类
    required float  weight = 2;// 权重
}

message UserBookInstCid3s
{
    required string uid = 1; //用户pin
    repeated BookCid3Inst cid3Inst = 2; //用户图书下感兴趣的三级分类
}
```
### pom 插件设置 ，配置说明看注释
```xml
 <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.7</version>
            <executions>
                <execution>
                    <id>compile-protoc</id>
                    <phase>generate-sources</phase>

                    <configuration>
                        <tasks>
                            <mkdir dir="target/generated-sources"/>
                            <path id="proto.path">
                                <!--proto文件定义的位置-->
                                <fileset dir="src/main/resources/proto">
                                    <include name="**/*.proto"/>
                                </fileset>
                            </path>
                            <pathconvert pathsep=" " property="proto.files"
                                         refid="proto.path"/>
                                <!--proto 执行文件 所在位置-->
                            <exec executable="E:\softs\protoc-2.5.0-win32\protoc.exe">
                                <arg value="--java_out=target/generated-sources"/>
                               <!-- 生成Java类文件 所在位置-->
                                <arg value="-I${project.basedir}/src/main/resources/proto"/>
                                <arg line="${proto.files}"/>
                            </exec>
                        </tasks>
                        <sourceRoot>target/generated-sources</sourceRoot>
                    </configuration>
                    <goals>
                        <goal>run</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
```
点击IDEA的compile 就可以生成相应的Java 类了。  
点击IDEA的deploy 部署到私服，供其他人使用。  

## 利用proto 自带执行文件生成


### protobuf 文件 
```xml
package p13.search; 


message BookCid3Inst
{
    required string cid3 = 1; //三级分类
    required float  weight = 2;// 权重
}

message UserBookInstCid3s
{
    required string uid = 1; //用户pin
    repeated BookCid3Inst cid3Inst = 2; //用户图书下感兴趣的三级分类
}
```
### protoc.exe 执行步骤
```xml

  1、 生成C代码  
 protoc.exe -proto_path=proto文件所在的目录 --cpp_out=生成C代码所在的位置
  2、 生成Java代码
 protoc.exe -proto_path=proto文件所在的目录 --java_out=生成java代码所在的位置
 
```
  
具体Linux下安装请参考：

http://blog.csdn.net/realxie/article/details/7456013   


Protobuf语言指南   

http://www.cnblogs.com/dkblog/archive/2012/03/27/2419010.html