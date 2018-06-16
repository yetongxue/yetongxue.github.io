---
title: 如何上传自己的jar包到maven公共远程中央仓库
date: 2018-03-16 14:48:41
tags: [Java,Maven]
categories: 
  - Java
  - Maven
---

本文是以上传`https://oss.sonatype.org/`maven中央仓库为例

# 注册账号
如果有账号可忽略该步骤，注册地址：`https://issues.sonatype.org`，这里的账号和密码一定要记住。

# 创建工单
**在首页最上面导航中点击`Create`创建工单：**
<!--more-->
![](http://ofbphtmkb.bkt.clouddn.com/201805/qqjie-tu-20180516105345.png)

**填写工单资料：**

![](http://ofbphtmkb.bkt.clouddn.com/201805/2.png)

- Project：选择开源项目
- Issue Type：选择创建新项目
- Summary：随意命名
- Group Id：唯一标识，我是用`com.qianxunclub`,因为这个是我自己的域名，管理员会问qianxunclub.com这个是不是自己的网站，回答是就好了，如果使用`com.github.xxxxx`之类的，会方便一些。
- Project URL：项目源码地址，如`https://gitee.com/qianxunclub/qianxunclub-maven-plugin`
- SCM url：项目管理地址，如：https://gitee.com/qianxunclub/qianxunclub-maven-plugin.git

**发布**

然后点击发布会成功创建一个工单，工单状态为：open

# 查看工单状态

**点击上方导航栏`Issue`选择自己的工单：**

![](http://ofbphtmkb.bkt.clouddn.com/201805/3.png)

等到审核状态为`RESOLVED `，恭喜你，审核已经成功，第一次审核要一天左右 ，因为时差原因，他们工作时间是我们的晚上，之后在创建工单如果GroupId 满足基本要求基本就是秒过。

![](http://ofbphtmkb.bkt.clouddn.com/201805/4.png)

# 上传jar包到maven中央仓库

**配置maven到`settings.xml`文件，添加以下内容：**

```
<servers> 
    <server> 
        <id>snapshots</id> 
        <username>https://issues.sonatype.org的注册账号</username> 
        <password>https://issues.sonatype.org的注册密码</password> 
    </server>
</servers>
```
> 这里注意以下，如果使用特殊符号，是需要转义的，例如：`pwd&`，密码要填写成`pwd&amp;`
> 
**在自己的项目中修改`pom.xml`，添加以下内容：**


```
<distributionManagement>
    <repository>
        <id>snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
    </repository>
    <snapshotRepository>
        <id>snapshots</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

**到这里就配置完成了，接下来开始发布试试了：**

```
mvn clean package deploy
```

出现`success`字样，代表已经成功，可以在`https://oss.sonatype.org/content/groups/public`找到自己的jar包了。