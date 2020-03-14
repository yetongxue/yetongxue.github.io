---
title: jvm-gc-jstat
date: 2020-03-14 14:36:39
tags: [Java,jvm]
categories: 
  - jvm
  - java
  - gc
  - jstat

---
<font color=red>注意！！！：使用的jdk版本是jdk8.</font>

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。命令的格式如下：

	jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]

类加载统计：