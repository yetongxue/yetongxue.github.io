---
title: Spring Cloud Config分布式配置中心的使用和遇到的坑
date: 2018-06-06 10:09:55
tags: [Java,SpringCloud,分布式]
categories:
  - Java
  - SpringCloud
---

# 分布式配置中心

为什么要有用分布式配置中心这玩意儿？现在这微服务大军已经覆盖了各种大小型企业，每个服务的粒度相对较小，因此系统中会出现大量的服务，每个服务都要有自己都一些配置信息，或者相同的配置信息，可能不同环境每个服务也有单独的一套配置，这种情况配置文件数量比较庞大，维护起来相当费劲，举个栗子：  
在开发的过程中，一般数据库是开发环境数据库，所有服务DB的IP配置为：92.168.0.1，突然老大说，开发环境换了，DB的IP要修改，这下可不好受了，所有模块挨个修改DB的配置，就问你难受不难受？  
这个时候分布式配置中心就发挥了很大的优势，只需要修改配置中心配置，所有服务即可自动生效，爽不爽！  

<!-- more -->

# Spring Cloud Config

官网地址：http://cloud.spring.io/spring-cloud-config/

## 简介

`Spring Cloud Config`为服务端和客户端提供了分布式系统的外部化配置支持。配置服务器为各应用的所有环境提供了一个中心化的外部配置。它实现了对服务端和客户端对`Spring Environment`和`PropertySource`抽象的映射，所以它除了适用于Spring构建的应用程序，也可以在任何其他语言运行的应用程序中使用。作为一个应用可以通过部署管道来进行测试或者投入生产，我们可以分别为这些环境创建配置，并且在需要迁移环境的时候获取对应环境的配置来运行。  

置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。当然他也提供本地化文件系统的存储方式。  

使用 spring Cloud 进行集中式配置管理，将以往的配置文件从项目中摘除后放到Git 或svn中集中管理，并在需要变更的时候，可以通知到各应用程序，应用程序刷新配置不需要重启。  
## 实现原理

其实这个实现原理相对比较简单一些，基于git的交互操作。

1. 我们把配置文件存放到git上面
2. Spring Cloud Config配置中心服务连接git
3. 客户端需要配置配置信息从配置中心服务获取
4. 当客户端启动，会从配置中心获取git上面的配置信息

# 配置中心服务端

**pom.xml添加依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <!-- spring cloud -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.SR3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**Application启动类添加注解**

添加`@EnableConfigServer`注解，启用配置中心：

```java
package com.qianxunclub;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;


/**
* @author chihiro.zhang
*/
@SpringBootApplication
@EnableConfigServer
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

**配置文件**

在`application.yml`或者`application.properties`添加配置信息：

```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/qianxunclub/spring-boot-config-repo
          default-label: master
          search-paths: /**
          basedir: target/config
```

- spring.cloud.config.server.git.uri：配置git仓库地址
- spring.cloud.config.server.git.search-paths：仓库文件夹目录，如果是`/**`，就是所有目录所有文件
- spring.cloud.config.server.git.default-label：配置仓库的分支
- spring.cloud.config.server.git.basedir：配置文件拉去到本地的目录位置

**启动测试**

首先在git里面添加一个`application-dev.yml`配置文件，内容如此下：
```yml
test: 我是配置中心配置信息
```

已经配置完成了，启动一波试试，看效果咋样，正常情况下是可以正常启动的，然后获取配置文件试试  
访问地址：http://localhost:8888/test/dev  
如果返回如下，就是成功了：
```json
{
    "name":"test",
    "profiles":[
        "dev"
    ],
    "label":null,
    "version":"64e7882a8f280641724e454a2db5a3da7b44d3d4",
    "state":null,
    "propertySources":[
        {
            "name":"https://gitee.com/qianxunclub/spring-boot-config-repo/application-dev.yml",
            "source":{
                "test":"配置中心的配置信息"
            }
        }
    ]
}
```

http请求地址和资源文件映射如下:

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

# 配置中心客户端使用

**pom.xml添加依赖**  
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <!-- spring cloud -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.SR3</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**配置文件**

创建`bootstrap.yml`文件，切记，是`bootstrap.yml`文件`bootstrap.yml`文件，我就因为写到了`application.yml`这个里面，各种出现问题啊，添加如下配置：
```yml
spring:
  cloud:
    config:
      name: application
      profile: dev
      label: master
      uri: http://localhost:8888/
```
- spring.cloud.config.label：指明远程仓库的分支
- spring.cloud.config.profile：指定不同环境配置文件，和git仓库的 `application-dev.yml`对应  
- spring.cloud.config.name：配置名称，一般和git仓库的`application-dev.yml`对应
- spring.cloud.config.uri：上面的配置中心服务地址  

**启动测试**


先添加一个获取配置信息的类:  
```java


/**
 * @author chihiro.zhang
 */
@Configuration
@EnableAutoConfiguration
public class DemoConfiguration {

    @Value("${test}")
    private  String test;
}

```
找个地方随便调用一下，输出这个test，就会打印上面git里面配置的信息了，爽不！


# 说说中间遇到的坑

1. 服务端git配置死活获取不了git仓库配置文件

```yml
spring:
    cloud:
        config:
        server:
            git:
                uri: https://gitee.com/qianxunclub/spring-boot-config-repo
                default-label: master
                search-paths: /**
                basedir: target/config
```

当时这个`uri`配置的是公司的git仓库，公司的git仓库访问是需要开代理才能有权限访问的，代理也开了，可是一直报错：

```
Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Wed Jun 06 11:10:56 CST 2018
There was an unexpected error (type=Not Found, status=404).
Cannot clone or checkout repository: http://xxx.com:5080/framework/config-repo
```
很郁闷，不知道为啥，可是就在刚刚，就刚刚，写博客的时候，有测试了一下，通了。。。。日了狗了，不知道啥原因，等研究出来了再来补充。

2. 客户端配置一定要配置在`bootstrap.yml`里面
`uri`默认会调用端口为`8888`的地址`http://localhost:8888/`  
启动的时候，会加载`label`和`uri`,`profile`配置，`profile`可以在启动参数添加，`profile`也可以加在`application.yml`添加  
`name`也可以加在`application.yml`添加  

# demo

配置中心服务端：https://gitee.com/qianxunclub/qianxunclub-springboot-config  
配置git仓库：https://gitee.com/qianxunclub/qianxunclub-springboot-config  
配置客户端使用：https://gitee.com/qianxunclub/qianxunclub-starter-demo  
客户端主要配置在：https://gitee.com/qianxunclub/qianxunclub-starter-parent/tree/master/qianxunclub-starter-config