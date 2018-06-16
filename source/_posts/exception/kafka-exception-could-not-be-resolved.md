---
title: kafka发送消息问题，could not be resolved
date: 2017-04-18 11:27:23
tags: [Exception,Kafka]
categories: 
  - 异常记录
---
# 异常描述
我用的阿里云服务器，我在服务器上面安装了一个kafka 
为啥不能发送的，百度了好多 说啥DNS不对.

`
2016/09/18 11:11:46 [error] 5720#0: [lua] producer.lua:258: buffered messages send to kafka err: iz9405kumw9z could not be resolved (3: Host not found), retryable: true, topic: lualog, partition_id: 0, length: 1, context: ngx.timer, clie 
nt: 183.12.65.116, server: 0.0.0.0:80
`
<!-- more -->
kafka配置文件（config/server.properties）： 

![](http://img.ask.csdn.net/upload/201609/18/1474168727_238864.png)

nginx日志： 
![](http://img.ask.csdn.net/upload/201609/18/1474168585_423649.png)


# 异常分析

从异常看，很明显是HOST访问不了，于是乎，要从Kfka网络配置入手了。

# 解决办法

![](http://img.ask.csdn.net/upload/201609/18/1474186554_681266.png)

使用下面这个地址 我写的是内网IP地址，换成外网的就可以正常使用了！