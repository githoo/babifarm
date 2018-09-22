+++
categories = ["技术"]
date = "2016-08-28T15:06:46-07:00"
draft = false
slug = ""
tags = ["思考"]
title = " Java内存模型设计思想 "

+++

## Java内存模型设计


多线程之间的数据通信 ：共享变量和消息机制

concurent

java内存模型使用的共享变量机制

ACTOR模型使用的是消息机制

主内存 进程级？
工作内存 线程级
多线程并发，即多核cpu执行

voliatile  实时更新cpu高速缓存(工作内存，线程使用的本地内存)

atomic  机制 cas

锁机制
voliatile atomic 变量，控制读写状态
 

主内存  与 工作内存之间的关系

工作内存的初始变量是从主内存copy过来

工作内存默认是批处理更新主内存共享变量

如果是voliatile 共享变量，工作内存更新，立即更新主内存共享变量

主内存存放共享变量 堆

多cpu 每个cpu有高速缓存 cpu执行指令后的结果

voliatile机制

store load 保证依赖关系的顺序 编译成字节码时优化处理
happen-before 原则 写在读前

atomic 机制

cas机制保证多线程安全
compare and set 更新前比较新旧值，如果不等就更新，否则循环尝试更新
