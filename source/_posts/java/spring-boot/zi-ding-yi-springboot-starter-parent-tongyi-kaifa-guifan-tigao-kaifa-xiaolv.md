---
title: 自定义SpringBoot的starter-parent，制定开发规范，和提高开发效率
date: 2018-05-25 10:40:00
tags: [Java,SpringBoot]
categories: 
  - Java
  - SpringBoot
---

# 写在前面

SpringBoot咱们开发一般`pom.xml`是都集成`spring-boot-starter-parent`的，这里为什么要自定义一个starter-parent呢，主要以下几个理由：  
1. 定义开发规范：自定义parent可以制定统一配置和依赖规范  
2. 提高开发效率 
3. 统一模块插件化管理：可以在parent实现需要的插件配置，如：redis、mysql、日志、参数教研等  
4. 项目版本升级和依赖包升级更加方便统一  
5. 日志收集等可以从切面获取各个项目数据和日志等
6. 项目持续集成部署交付方便处理  

<!-- more -->

 其实好处挺多的，谁用谁知道。

# 一、准备

*`qianxunclub-starter-parent`里面已经开发好的几个插件，需要依据公司真实环境修改配置*

源码地址：  
```
git clone https://gitee.com/qianxunclub/qianxunclub-starter-parent.git
```

因为`qianxunclub-starter-parent`依赖了另一个`framework-common`工具包，所以要下载该包源码并且编译：  
```
git clone https://gitee.com/qianxunclub/framework-common.git
 ```

 ```
cd framework-common
```

```
mvn clean install
```

# 二、starter-parent插件功能列表

本starter已包含一下功能：  

1. [lombok插件添加](https://projectlombok.org/features/all)
2. [日志输出：qianxunclub-starter-logging](#qianxunclub-starter-logging)
3. [swagger接口文档：qianxunclub-starter-swagger](#qianxunclub-starter-swagger)
4. [跨域请求、http编码配置入参校验等：qianxunclub-starter-web](#qianxunclub-starter-web)
5. [mysql以及MybatisPlus引用：qianxunclub-starter-mysql](#qianxunclub-starter-mysql)（如果不使用，禁止引入该包，否则项目启动出错）
6. [redis支持：qianxunclub-starter-redis](#qianxunclub-starter-redis)（如果不使用，禁止引入该包，否则项目启动出错）  

# 三、开发配置

## pom.xml编辑

POM需要依赖父级`qianxunclub-starter-parent`：  
```xml
<parent>
	<groupId>com.qianxunclub</groupId>
	<artifactId>qianxunclub-starter-parent</artifactId>
	<version>0.0.1-SNAPSHOT</version>
</parent>
```

继承该依赖，无需再引用`spring-boot-starter-parent`,本项目已经继承的`spring-boot-starter-parent`版本为`1.5.9.RELEASE`。

## 项目目录结构

```
qianxunclub-demo
├── pom.xml
├── qianxunclub-demo.iml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── qianxunclub
    │   │           ├── Application.java
    │   │           └── demo
    │   │               └── web
    │   └── resources
    │       └── application.yml
    └── test
        └── java
```

1. com.qianxunclub：项目根包名，必须以该名称命名，并且SpringBoot启动类必须在该目录
2. group：项目分组名称，通常依据项目在GIT分组名称命名
3. demo：项目名称，通常依据GIT名称命名
4. web：controller控制器，，必须以该名称命名
5. resources：资源目录


## application.yml基础配置信息

```yml
app:
  group: group
  name: demo
  descriptions: 项目描述
  author: 千寻啊千寻
  email: 960339491@qq.com
spring:
  profiles:
    include:
    - web
    - swagger
    - logging
    - mysql
```

1. group：项目分组名称，和包结构中的分组名称必须一致  
2. name：项目名称。和包结构中项目名称必须一致  
3. descriptions：项目描述信息  
4. author：项目负责人名称  
5. email：项目负责人邮箱  
6. spring.profiles.include：使用已经定义的starter功能,项目参考本文第三章  


# 四、starter使用说明

## 日志输出：qianxunclub-starter-logging

<span id = "qianxunclub-starter-logging">　</span>

### 配置方式

#### pom.xml引入包：
```
<!-- 日志配置信息 -->
<dependency>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-logging</artifactId>
</dependency>
```

#### application.yml配置

profiles需要引入配置：

```yml
spring:
  profiles:
    include:
    - logging
```

自定义配置：

```yml
logging:
  level:
    com.qianxunclub: debug
```

以下为缺省配置：

```yml
logging:
  level:
    org.springframework: info
    com.qianxunclub: info
  pattern:
    console: "%date{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{64})  - %msg%n"
  file: /logs/${app.group:}/${app.name:${spring.application.name:application}}/${spring.application.name:application}.log
```

**
日志默认存放目录：/logs/项目分组/项目名称/日志文件
如果没有项目分组：/logs/项目名称/日志文件
**


### 使用方式

只需要在类上面添加`@Slf4j`注解，即可使用`log`对象打印日志

```java
@Slf4j
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
		log.info("日志信息");
	}
}
```

## swagger接口文档：qianxunclub-starter-swagger

官网：https://swagger.io/

<span id = "qianxunclub-starter-swagger">　</span>

### 配置方式

#### pom.xml引入包：

```xml
<!-- 自动生成接口文档 -->
<dependency>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-swagger</artifactId>
</dependency>
```

#### application.yml配置

swagger默认是不启用的，必须添加一下配置，才可以开启配置

```yml
app:
  swagger: true
spring:
  profiles:
    include:
    - swagger
```

#### 启动类添加`@EnableSBCSwagger`注解，开启接口

```java
@EnableSBCSwagger
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

### 使用方式

启动项目成功后,访问API接口地址，例如：http://localhost:8080/swagger-ui.html。



## mysql以及MybatisPlus引用：qianxunclub-starter-mysql

<span id = "qianxunclub-starter-mysql">　</span>

### 3.4.1配置方式

#### pom.xml引入包：

```xml
<!-- MYSQL配置信息 -->
<dependency>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-mysql</artifactId>
</dependency>
```

#### application.yml配置

引入mysql配置：

```yml
spring:
  profiles:
    include:
    - mysql
```

添加数据源信息：

```yml
app:
  group: group
  name: oneway
  description: demo
  author: 千寻啊千寻
  email: 960339491@qq.com
  datasource:
    host: xxx.xxx.xxx.xxx
    username: root
    password: xxx
    druid:
      public-key: xxx

```
端口默认为：3306  
数据库密码加密：

```
java -cp druid-1.1.6.jar com.alibaba.druid.filter.config.ConfigTools your_password
```

### 使用方式

以上配置完成后即可使用mysql以及MybatisPlus，该starter已封装部分常用数据库操作，方法如下：  
MybatisPlus使用说明：http://mp.baomidou.com/#/?id=%e7%ae%80%e4%bb%8b  

**1、Entity规范**  

数据库对应的Entity需要继承`BaseEntity`，如下：

```java
@Data
@TableName("t_app")
public class AppEntity extends BaseEntity {

}
```
**2、Param规范**  

Param为入参查询常用发放，Param需要继承`BaseParam`,如下：

```java
@Data
public class AppParam extends BaseParam {

}
```

**3、Mapper规范**  

需要添加`@Mapper`注解，继承`BaseMapper`，并且要指定Entity泛型，如下：

```java
@Mapper
public interface AppMapper extends BaseMapper<AppEntity> {
}
```

**4、Dao规范**  

需要继承`BaseDao`,并且指定泛型，如下：  

```java
@Component
public class AppDao extends BaseDao<AppMapper,AppEntity,AppParam> {

}

```

**5、mapper.xml规范**  

默认可以不添加mapper.xml，如果遇到特定的SQL，可在`resources`资源目录添加mapper文件夹，并添加对应的XML即可


## redis支持：qianxunclub-starter-redis

<span id = "qianxunclub-starter-redis">　</span>

### 配置方式

#### pom.xml引入包：

```xml
<!-- MYSQL配置信息 -->
<dependency>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-redis</artifactId>
</dependency>
```

#### application.yml配置

引入mysql配置：

```yml
spring:
  profiles:
    include:
    - redis
```

添加redis配置信息：

```yml
spring:
  redis:
    host: 120.25.173.32
    password: 123456

```

端口默认为：6379  

下面为缺省配置：

```yml
spring:
  redis:
    database: 0
    host: localhost
    port: 6379
    password: 
    pool:
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池中的最大空闲连接
      max-idle: 8
      # 连接池中的最小空闲连接
      min-idle: 0
      # 连接超时时间（毫秒）
      timeout: 0
```

### 使用方式

在类中可直接注入`RedisTemplate`或`StringRedisTemplate`即可使用，例如：

```java
@Slf4j
@RestController
@RequestMapping(value = "/redis")
@AllArgsConstructor
@Api(description = "redis")
public class RedisController {

	private RedisTemplate redisTemplate;
	private StringRedisTemplate stringRedisTemplate;



	@ApiOperation("add")
	@ResponseBody
	@RequestMapping(value="add",method = RequestMethod.GET)
	public void add() {
		ValueOperations<String, String> operations=redisTemplate.opsForValue();
		operations.set("ValueOperations","1");
		log.info(operations.get("ValueOperations"));

		stringRedisTemplate.opsForValue().set("aaa", "111");
		log.info(stringRedisTemplate.opsForValue().get("aaa"));
	}

}
```


## 跨域请求、http编码配置入参校验等：qianxunclub-starter-web

<span id = "qianxunclub-starter-web">　</span>

### 配置方式

#### pom.xml引入包：

```xml
<!-- 支持跨域请求 -->
<dependency>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-web</artifactId>
</dependency>
```

#### application.yml配置 

profiles需要引入配置：

```yml
spring:
  profiles:
    include:
    - web
```

# 五、配置最终效果如下

pom.xml：

```xml
<modelVersion>4.0.0</modelVersion>

<groupId>com.qianxunclub</groupId>
<artifactId>qianxunclub-demo</artifactId>
<version>1.0-SNAPSHOT</version>

<parent>
    <groupId>com.qianxunclub</groupId>
    <artifactId>qianxunclub-starter-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>

<dependencies>

    <!-- 自动生成接口文档 -->
    <dependency>
        <groupId>com.qianxunclub</groupId>
        <artifactId>qianxunclub-starter-swagger</artifactId>
    </dependency>
    <!-- web配置信息 -->
    <dependency>
        <groupId>com.qianxunclub</groupId>
        <artifactId>qianxunclub-starter-web</artifactId>
    </dependency>
    <!-- 日志配置信息 -->
    <dependency>
        <groupId>com.qianxunclub</groupId>
        <artifactId>qianxunclub-starter-logging</artifactId>
    </dependency>
    <!-- MYSQL配置信息 -->
    <dependency>
        <groupId>com.qianxunclub</groupId>
        <artifactId>qianxunclub-starter-mysql</artifactId>
    </dependency>
</dependencies>
```

application.yml:

```yml
app:
  group: group
  name: demo
  description: demo服务
  author: 千寻啊千寻
  email: 960339491@qq.com
  swagger: true
  datasource:
    host: xxx.xxx.xxx.xxx
    username: root
    password: xxx
    druid:
      public-key: xxx

spring:
  profiles:
    include:
    - web
    - swagger
    - logging
    - mysql
```


完！