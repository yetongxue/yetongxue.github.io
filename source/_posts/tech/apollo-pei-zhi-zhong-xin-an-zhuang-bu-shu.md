---
title: Apollo分布式配置中心部署以及使用
date: 2018-06-12 11:18:00
tags: [Java,分布式,SpringBoot,Spring Cloud]
categories: 
  - 技术杂文
---

# 一、简介

>Apollo（阿波罗）是携程框架部门研发的分布式配置中心，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

官方github：https://github.com/ctripcorp/apollo  

作者对Apollo对介绍：
https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D  

<!--more-->

# 二、安装部署

## 基础设施

本次部署环境为DEV（开发环境）、FAT（测试环境）、UAT（预生产）、PRO（生产）  

**应用服务器：**

| 环境 | 服务器 | 服务 | 端口 |
| :---: | --- | --- | :---: |
| / | 192.168.35.206 | apollo-portal | 9102 |
| DEV | 192.168.35.207 | apollo-configservice<br/>apollo-adminservice | 9100<br/>9101 |
| FAT | 192.168.35.208 | apollo-configservice<br/>apollo-adminservice | 9100<br/>9101 |
| UAT | 192.168.35.209 | apollo-configservice<br/>apollo-adminservice | 9100<br/>9101 |
| PRO | 192.168.35.210 | apollo-configservice<br/>apollo-adminservice | 9100<br/>9101 |

**数据库服务器：**

| 环境 | 服务器 | 服务 | 数据库 | 端口 |
| :---: | --- | --- | --- | :---: |
| / | 192.168.35.226 | apollo-portal | ApolloPortalDB | 3306 |
| DEV | 192.168.35.227 | apollo-configservice<br/>apollo-adminservice | ApolloConfigDB | 3306 |
| FAT | 192.168.35.228 | apollo-configservice<br/>apollo-adminservice | ApolloConfigDB | 3306 |
| UAT | 192.168.35.229 | apollo-configservice<br/>apollo-adminservice | ApolloConfigDB | 3306 |
| PRO | 192.168.35.230 | apollo-configservice<br/>apollo-adminservice | ApolloConfigDB | 3306 |

##  配置

**下载代码：**  

```
git clone https://github.com/ctripcorp/apollo.git
```

比较重要的几个项目：  
- apollo-configservice：提供配置获取接口，提供配置更新推送接口，接口服务对象为Apollo客户端  
- apollo-adminservice：提供配置管理接口，提供配置修改、发布等接口，接口服务对象为Portal，以及Eureka  
- apollo-portal：提供Web界面供用户管理配置  
- apollo-client：Apollo提供的客户端程序，为应用提供配置获取、实时更新等功能  

![](http://pa74vwr85.bkt.clouddn.com/release-message-notification-design.png)

上图简要描述了配置发布的大致过程：

- 用户在Portal操作配置发布
- Portal调用Admin Service的接口操作发布
- Admin Service发布配置后，发送ReleaseMessage给各个Config Service
- Config Service收到ReleaseMessage后，通知对应的客户端

**数据库初始化：**  

*下面的sql为大写格式，注意数据库的大小写敏感设置*

- ApolloPortalDB：执行`apollo\scripts\sql\apolloportaldb.sql`
- ApolloConfigDB：DEV FAT UAT PRO 环境执行`apollo\scripts\sql\apolloconfigdb.sql`

**调整配置并打包：**  

在`Apollo`项目中找到目录`apollo\scripts\`的配置文件`build.sh`或者`build.bat`  
1. 数据库配置  
修改数据库配置，上面的是`ApolloConfigDB`配置，下面的是`ApolloPortalDB`配置：

```
# apollo config db info  该数据库配置只需要配置一次，不同环境无需修改
apollo_config_db_url=jdbc:mysql://192.168.35.227:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=XXXX
apollo_config_db_password=XXXX

# apollo portal db info  该数据库依据不同环境配置对应的数据库连接，并且需要多次打对应环境的JAR包
apollo_portal_db_url=jdbc:mysql://192.168.35.226:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=XXXX
apollo_portal_db_password=XXXX
```

- apollo config db info  该数据库配置只需要配置一次，不同环境无需修改
- apollo portal db info  该数据库依据不同环境配置对应的数据库连接，并且需要多次打


2. 修改环境调用地址  

```
# meta server url, different environments should have different meta server addresses
dev_meta=http://192.168.35.207:9100
fat_meta=http://192.168.35.208:9100
uat_meta=http://192.168.35.209:9100
pro_meta=http://192.168.35.210:9100
```

3. 修改数据库数据  

在DEV FAT UAT PRO 对应的`ApolloConfigDB`数据库中,找到表`ServerConfig`中的`eureka.service.url`配置项：

```sql
UPDATE apolloconfigdb.ServerConfig SET ServerConfig.`Value`='http://localhost:9100/eureka/' WHERE `Key`='eureka.service.url';
```

修改环境配置，在`ApolloPortalDB`数据库修改表`ServerConfig`中的`apollo.portal.envs`:

```sql
UPDATE apolloportaldb.serverconfig SET serverconfig.`Value`='dev,fat,uat,pro' WHERE `Key`='apollo.portal.envs';
```

具体`eureka`配置，可以查看官网：https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97

4. 编译、打包

```
./build.sh
```

- 该脚本会依次打包`apollo-configservice`, `apollo-adminservice`, `apollo-portal`和`apollo-client`。
- 由于`ApolloConfigDB`在每个环境都有部署，所以对不同环境的`config-service`和`admin-service`需要使用不同的数据库连接信息打不同的包，`portal`和`client`只需要打一次包即可

##  开始部署

**部署程序到对应的服务器**  

1. 部署apollo-configservice

将`apollo-configservice/target/`目录下的`apollo-configservice-x.x.x-github.zip`上传到服务器上，解压  
修改`scripts/startup.sh` ： 

```
LOG_DIR=/opt/logs/100003171
SERVER_PORT=9100
```

执行`scripts/startup.sh`即可  
如需停止服务，执行`scripts/shutdown.sh.`  


2. 部署apollo-adminservice

将`apollo-adminservice/target/`目录下的`apollo-adminservice-x.x.x-github.zip`上传到服务器上，解压  
修改`scripts/startup.sh`：

```
LOG_DIR=/opt/logs/100003172
SERVER_PORT=9101
```

执行`scripts/startup.sh`即可  
如需停止服务，执行`scripts/shutdown.sh.`  

3. 部署apollo-portal

将`apollo-portal/target/`目录下的`apollo-portal-x.x.x-github.zip`上传到服务器上，解压  
修改`scripts/startup.sh`：

```
LOG_DIR=/opt/logs/100003173
SERVER_PORT=9102
```

执行`scripts/startup.sh`即可  
如需停止服务，执行`scripts/shutdown.sh.`  

**访问测试**  

上面部署完成，可以测试

访问不同环境的`eureka`，查看服务注册情况是否正确：

```
http://192.168.35.207:9100/
```

如果可以看到:

```
192.168.35.207:apollo-adminservice:9101
192.168.35.207:apollo-configservice:9100
```

两个服务都为UP，正常！

访问客户端：

```
http://192.168.35.206:9102/
```

登录，默认用户名密码为：`apollo/admin`  

新建项目测试。

# 三、使用配置中心配置信息

1. maven引入上面步骤编译打包成功的`apollo-core`和`apollo-client`包：  

```xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-core</artifactId>
    <version>0.11.0-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>0.11.0-SNAPSHOT</version>
</dependency>
```

2. 创建app.properties  

请确保classpath:/META-INF/app.properties文件存在，并且其中内容为自己的项目名称，而且要保持唯一：
```
app.id=demo
```

3. 环境变量配置  

**本地开发**  

如果是本地开发，可以在开发工具添加Environment：

```
env=DEV
```

**线上环境配置方式：**  
- 使用Java启动参数添加`java -Denv=YOUR-ENVIRONMENT -jar xxx.jar`
- 通过操作系统的System Environment
- 通过配置文件：  
对于Mac/Linux，文件位置为/opt/settings/server.properties  
对于Windows，文件位置为C:\opt\settings\server.properties  

4. 配置apollo-env.properties  

在项目中引用`apollo-core`和`apollo-client`包，在`apollo-core`包中可以看到`apollo-env.properties`配置文件，默认配置为打包前配置的信息：

```
local.meta=http://localhost:8080
dev.meta=http://192.168.35.207:9100
fat.meta=http://192.168.35.208:9100
uat.meta=http://192.168.35.209:9100
lpt.meta=${lpt_meta}
pro.meta=http://192.168.35.210:9100
```

如果需要修改或者覆盖的话，在项目的`resources`从上面复制一个`apollo-env.properties`文件，修改对应环境信息就可以了

5. 启用配置  
在启动类添加`@EnableApolloConfig`注解即可：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;


/**
 * @author chihiro.zhang
 */
@EnableApolloConfig
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

5. 测试  

添加一个测试的类`DemoConfiguration`，当然配置中心要有下面配置的配置信息:
```
@Configuration
@EnableAutoConfiguration
public class DemoConfiguration {
    @Value("${demo}")
    private String demo;
}
```

完成！

# 四、部署方案

这个图是计划部署的方案，并不是上面写的例子的方案

![](http://pa74vwr85.bkt.clouddn.com/Apollo%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E4%B8%AD%E5%BF%83.png)



