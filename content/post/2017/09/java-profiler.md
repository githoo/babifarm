+++
categories = ["工具"]
date = "2017-09-09T10:06:46-07:00"
draft = false
slug = ""
tags = ["java","profiler"]
title = "java profiler tool"

+++


## YourKit Java Profiler

Yourkit是一款集Java profile数据收集及分析的工具，常用于分析Java程序性能瓶颈，带有cpu prifling、memory profiling等等各种profiling。以下会从yourkit使用场景、监控快照数据收集及分析、远程Java进程实时监控三方面来介绍。



### 收集Java profile快照数据

1、yourkit下载：
window查看：https://www.yourkit.com/download/YourKit-JavaProfiler-2017.02-b66.exe
Linux服务端插件：https://www.yourkit.com/download/YourKit-JavaProfiler-2017.02-b66.zip

2、应用程序执行脚本增加：
```
-agentpath:/export/servers/java_profile/yjp-2017.02/bin/linux-x86-64/libyjpagent.so：
```
eg:
```
java -agentpath:/export/servers/java_profile/yjp-2017.02/bin/linux-x86-64/libyjpagent.so -jar /export/servers/proxy/proxy-1.0-SNAPSHOT.jar
```

3、收集数据：
```
 java -jar /export/servers/java_profile/yjp-2017.02/lib/yjp-controller-api-redist.jar localhost 10001 start-cpu-sampling
 java -jar /export/servers/java_profile/yjp-2017.02/lib/yjp-controller-api-redist.jar localhost 10001 stop-cpu-profiling
 java -jar /export/servers/java_profile/yjp-2017.02/lib/yjp-controller-api-redist.jar localhost 10001 capture-performance-snapshot
```

4、查看分析
打开yourkit，File-->open snapshot，选中上一步down下的snapshot文件，如图可看出代码中哪些方法比较占CPU。还可看thread繁忙状态、内存使用、性能曲线中net io等。

### 远程调试
远程连接Java进程，实时查看其cpu、内存等监控（类似于visualVm，但比visualVm要详细）。Yourkit能做的事情并不止这些，其他类型的监控可参考官网介绍：https://www.yourkit.com/docs/java/help/。

yourkit软件 welcome page点击connect to remote application，按照以下格式输入：
跳板机:跳板机port Java应用服务器:监控收集端口，点击connect
远程调试
10.188.188.120:80 10.10.130.116:10001

应用服务器端执行：
/export/servers/java_profile/yjp-2017.02/bin/yjp.sh -attach 

参考：https://www.yourkit.com/docs/java/help/attach_wizard.jsp



### yourKit command  
YourKit Java Profiler2016.02-b36 command line tools
```
Usage: java -jar yjp-controller-api-redist.jar <host> <port> <command>
where <command> is one of:
status
capture-memory-snapshot
capture-hprof-snapshot
capture-performance-snapshot
start-cpu-sampling
start-cpu-tracing
start-cpu-call-counting
stop-cpu-profiling
clear-cpu-data
start-alloc-recording-all
  // record all objects
start-alloc-recording-adaptive [alloc-sampled]
  // record all objects with size >= 4 KB, and only each 10th smaller object
stop-alloc-recording
clear-alloc-data
start-monitor-profiling
stop-monitor-profiling
clear-monitor-data
enable-stack-telemetry
disable-stack-telemetry
force-gc

Examples:
java -jar yjp-controller-api-redist.jar localhost 10001 capture-memory-snapshot
java -jar yjp-controller-api-redist.jar localhost 10001 start-cpu-sampling
```

