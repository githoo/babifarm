
+++
categories = ["技术"]
date = "2016-08-02T13:06:46-07:00"
draft = false
slug = ""
tags = ["flume"]
title = "flume load balance 验证"

+++

  



1、验证说明

	数据从客户端发送给服务端，服务端进行负载均衡，发给两个处理节点：
	
								   |---------->node1---->show
	---test --> client -->server-->|	 
								   |---------->node2---->show


2、安装flume:

    download ：http://flume.apache.org/download.html
         tar -xvf apache-flume-1.6.0-bin.tar.gz
    修改conf/flume-env.sh配置文件,设置JAVA_HOME变量

3、测试环境：
	
	在一台服务器上进行测试

4、相关配置：
	
	在路径conf/simple_load下新建如下文件
      client.conf
    # Define a memory channel called c1 on a0 
    a0.channels.c1.type = memory 
    a0.channels.c1.capacity = 1000
    a0.channels.c1.transactionCapacity = 100   


    a0.sources.r1.channels = c1 
    a0.sources.r1.type = netcat
    a0.sources.r1.bind = localhost
    a0.sources.r1.port = 43210

    #a0.sinks.k1.type = logger 
    a0.sinks.k1.type = avro   
    a0.sinks.k1.channel = c1 
    a0.sinks.k1.hostname = localhost
    a0.sinks.k1.port = 43211 

    a0.channels = c1 
    a0.sources = r1 
    a0.sinks = k1 

    server.conf
    #Define a memory channel called c1 on a1 
    a1.channels = c1 
    a1.sources = r1 
    a1.sinks = k1 k2 
    a1.sinkgroups = g1 
    a1.sinkgroups.g1.sinks = k1 k2 
    a1.sinkgroups.g1.processor.type = load_balance 
    a1.sinkgroups.g1.processor.selector = round_robin 
    a1.sinkgroups.g1.processor.backoff = true 

    a1.channels.c1.type = memory   

    a1.sources.r1.channels = c1 
    a1.sources.r1.type = avro 
    a1.sources.r1.bind = 0.0.0.0 
    a1.sources.r1.port = 43211 

    a1.sinks.k1.channel = c1 
    a1.sinks.k1.type = avro 
    a1.sinks.k1.hostname = localhost
    a1.sinks.k1.port = 43212 
    a1.sinks.k2.channel = c1 
    a1.sinks.k2.type = AVRO 
    a1.sinks.k2.hostname = localhost
    a1.sinks.k2.port = 43213
    node1.conf
    #Define a memory channel called c1 on a2 
    a2.channels = c1 
    a2.sources = r1 
    a2.sinks = k1 
    a2.channels.c1.type = memory   

    a2.sources.r1.channels = c1 
    a2.sources.r1.type = avro 
    a2.sources.r1.bind = 0.0.0.0 
    a2.sources.r1.port = 43212

    a2.sinks.k1.channel = c1 
    a2.sinks.k1.type = logger 
    node2.conf
    #Define a memory channel called c1 on a3 
    a3.channels = c1 
    a3.sources = r1 
    a3.sinks = k1 
    a3.channels.c1.type = memory   

    a3.sources.r1.channels = c1 
    a3.sources.r1.type = avro 
    a3.sources.r1.bind = 0.0.0.0 
    a3.sources.r1.port = 43213

    a3.sinks.k1.channel = c1 
    a3.sinks.k1.type = logger 

5、测试命令：

     打开5个终端，分别顺序执行，node1,node2,server,client ,test
     具体命令如下：

	bin/flume-ng agent -n a3 -c conf/ -f conf/simple_load/node2.conf  -Dflume.root.logger=INFO,console
	
	2016-07-10 03:29:48,740 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:94)] Event: { headers:{} body: 31 31 31                                        111 }
	2016-07-10 03:29:48,741 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:94)] Event: { headers:{} body: 32 32 32                                        222 }
	
	bin/flume-ng agent -n a2 -c conf/ -f conf/simple_load/node1.conf  -Dflume.root.logger=INFO,console
	
	2016-07-10 03:29:51,042 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:94)] Event: { headers:{} body: 33 33 33                                        333 }
	2016-07-10 03:29:51,043 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:94)] Event: { headers:{} body: 34 34 34                                        444 }
	
	bin/flume-ng agent -n a1 -c conf/ -f conf/simple_load/server.conf -Dflume.root.logger=INFO,console
	bin/flume-ng agent -n a0 -c conf/ -f conf/simple_load/client.conf -Dflume.root.logger=INFO,console

	nc localhost 43210
	111
	ok
	222
	ok
	333
	ok
	444
	ok

6、测试结论：

	测试端发送4条消息，经过server负载处理，node1和node2分别各处理2条消息，达到预期。
