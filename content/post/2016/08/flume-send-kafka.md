+++
categories = ["技术"]
date = "2016-08-03T13:06:46-07:00"
draft = false
slug = ""
tags = ["flume"]
title = " flume 整合 kafka "

+++

## 一.kafka 安装 

1、下载http://kafka.apache.org/downloads.html

2、解压tar -zxvf kafka_2.10-0.8.1.1.tgz

3、启动服务

	1.首先启动zookeeper服务
	
	bin/zookeeper-server-start.sh config/zookeeper.properties
	
	2.启动Kafka

	bin/kafka-server-start.sh config/server.properties >/dev/null 2>&1 &
	
	3.创建topic创建一个"test"的topic，一个分区一个副本

	bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

	4.查看主题

	bin/kafka-topics.sh --list --zookeeper localhost:2181

	5.查看主题详情

	bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test

	6.删除主题

	bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic test

## 二. flume 整合 kafka  

1、安装flume:

    download ：http://flume.apache.org/download.html
    tar -xvf apache-flume-1.6.0-bin.tar.gz
    修改conf/flume-env.sh配置文件,设置JAVA_HOME变量

2、启动flume:

	$ cd  apache-flume-1.6.0-bin
	$ bin/flume-ng agent -n producer -c conf/ -f conf/test-kafka-sink.conf  -DFlume.root.logger=INFO,console

3、启动kafka消费

	$ ./kafka-console-consumer.sh  --zookeeper localhost:2181-topic test
	123

4、测试发送消息给flume source

	nc localhost 8285
	123
	ok



5、配置文件 test-kafka-sink.conf

	```
	
	#memory channel called ch1 on agent1
	producer.channels.channel1.type = memory
	
	# Define an Avro source called avro-source1 on agent1 and tell it
	# to bind to 0.0.0.0:41414. Connect it to channel ch1.
	producer.sources.source1.channels = channel1
	#producer.sources.source1.type = syslogudp
	producer.sources.source1.type = netcat
	producer.sources.source1.bind = 127.0.0.1
	producer.sources.source1.port = 8285
	
	# Define a logger sink that simply logs all events it receives
	# and connect it to the other end of the same channel.
	producer.sinks.sink1.channel = channel1
	
	producer.sinks.sink1.type = org.apache.flume.sink.kafka.KafkaSink
	producer.sinks.sink1.brokerList=127.0.0.1:9092
	producer.sinks.sink1.topic=test
	producer.sinks.sink1.batchSize=20
	
	
	
	# Finally, now that we've defined all of our components, tell
	# agent1 which ones we want to activate.
	producer.channels = channel1
	producer.sources = source1
	producer.sinks = sink1
	
	```