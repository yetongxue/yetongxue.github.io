---
title: SpringBoot启动Tomcat失败：Unable to start embedded Tomcat
date: 2018-05-22 10:40:00
tags: [Exception,SpringBoot]
categories: 
  - 异常记录
---

# 异常描述

>之前项目是dubbo的，分为两个服务，service数据层和web请求控制，使用dubbo互相调用的，现在要把dubbo去掉，使用SpringCloud的eureka了，要把两个合并成一个项目，这里并不是要把所有代码复制过去，而是把service使用maven引用到web项目里面，然后修改对应的配置和包路劲，启动的时候，竟然报错了，编译是没有任何问题的。

<!--more-->

```
10:02:26.349 logback [main] INFO  o.a.catalina.core.StandardEngine - Starting Servlet Engine: Apache Tomcat/8.5.23
10:02:26.463 logback [Tomcat-startStop-1] ERROR o.apache.catalina.core.ContainerBase - A child container failed during start
java.util.concurrent.ExecutionException: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Tomcat].StandardHost[localhost].TomcatEmbeddedContext[]]
        at java.util.concurrent.FutureTask.report(Unknown Source)
        at java.util.concurrent.FutureTask.get(Unknown Source)
        at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:939)
        at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:872)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1419)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1409)
        at java.util.concurrent.FutureTask.run(Unknown Source)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
        at java.lang.Thread.run(Unknown Source)
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Tomcat].StandardHost[localhost].TomcatEmbeddedContext[]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        ... 6 common frames omitted
Caused by: org.apache.catalina.LifecycleException: Failed to start component [Pipeline[StandardEngine[Tomcat].StandardHost[localhost].TomcatEmbeddedContext[]]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5117)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 6 common frames omitted
Caused by: org.apache.catalina.LifecycleException: Failed to start component [org.apache.catalina.authenticator.NonLoginAuthenticator[]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.core.StandardPipeline.startInternal(StandardPipeline.java:182)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 8 common frames omitted
Caused by: java.lang.NoSuchMethodError: javax.servlet.ServletContext.getVirtualServerName()Ljava/lang/String;
        at org.apache.catalina.authenticator.AuthenticatorBase.startInternal(AuthenticatorBase.java:1141)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 10 common frames omitted
10:02:26.464 logback [main] ERROR o.apache.catalina.core.ContainerBase - A child container failed during start
java.util.concurrent.ExecutionException: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Tomcat].StandardHost[localhost]]
        at java.util.concurrent.FutureTask.report(Unknown Source)
        at java.util.concurrent.FutureTask.get(Unknown Source)
        at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:939)
        at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:262)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.core.StandardService.startInternal(StandardService.java:422)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:793)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.startup.Tomcat.start(Tomcat.java:367)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer.initialize(TomcatEmbeddedServletContainer.java:99)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer.<init>(TomcatEmbeddedServletContainer.java:84)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory.getTomcatEmbeddedServletContainer(TomcatEmbeddedServletContainerFactory.java:554)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory.getEmbeddedServletContainer(TomcatEmbeddedServletContainerFactory.java:179)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.createEmbeddedServletContainer(EmbeddedWebApplicationContext.java:164)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.onRefresh(EmbeddedWebApplicationContext.java:134)
        at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:537)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
        at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
        at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107)
        at com.gwghk.base.platform.dfs.Application.main(Application.java:22)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Unknown Source)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
        at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Tomcat].StandardHost[localhost]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1419)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1409)
        at java.util.concurrent.FutureTask.run(Unknown Source)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
        at java.lang.Thread.run(Unknown Source)
Caused by: org.apache.catalina.LifecycleException: A child container failed during start
        at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:948)
        at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:872)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 6 common frames omitted
10:02:26.465 logback [main] WARN  o.s.b.c.e.AnnotationConfigEmbeddedWebApplicationContext - Exception encountered during context initialization - cancelling refresh attempt: org.springframework.context.ApplicationContextException: Unable to start embedded conta
iner; nested exception is org.springframework.boot.context.embedded.EmbeddedServletContainerException: Unable to start embedded Tomcat
10:02:26.485 logback [main] INFO  o.s.b.a.l.AutoConfigurationReportLoggingInitializer -

Error starting ApplicationContext. To display the auto-configuration report re-run your application with 'debug' enabled.
10:02:26.492 logback [main] ERROR o.s.boot.SpringApplication - Application startup failed
org.springframework.context.ApplicationContextException: Unable to start embedded container; nested exception is org.springframework.boot.context.embedded.EmbeddedServletContainerException: Unable to start embedded Tomcat
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.onRefresh(EmbeddedWebApplicationContext.java:137)
        at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:537)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
        at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
        at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107)
        at com.gwghk.base.platform.dfs.Application.main(Application.java:22)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
        at java.lang.reflect.Method.invoke(Unknown Source)
        at org.springframework.boot.loader.MainMethodRunner.run(MainMethodRunner.java:48)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:87)
        at org.springframework.boot.loader.Launcher.launch(Launcher.java:50)
        at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: org.springframework.boot.context.embedded.EmbeddedServletContainerException: Unable to start embedded Tomcat
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer.initialize(TomcatEmbeddedServletContainer.java:123)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer.<init>(TomcatEmbeddedServletContainer.java:84)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory.getTomcatEmbeddedServletContainer(TomcatEmbeddedServletContainerFactory.java:554)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory.getEmbeddedServletContainer(TomcatEmbeddedServletContainerFactory.java:179)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.createEmbeddedServletContainer(EmbeddedWebApplicationContext.java:164)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.onRefresh(EmbeddedWebApplicationContext.java:134)
        ... 16 common frames omitted
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardServer[-1]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.startup.Tomcat.start(Tomcat.java:367)
        at org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer.initialize(TomcatEmbeddedServletContainer.java:99)
        ... 21 common frames omitted
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardService[Tomcat]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:793)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 23 common frames omitted
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Tomcat]]
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
        at org.apache.catalina.core.StandardService.startInternal(StandardService.java:422)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 25 common frames omitted
Caused by: org.apache.catalina.LifecycleException: A child container failed during start
        at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:948)
        at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:262)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        ... 27 common frames omitted

```

# 异常分析

刚开始有点蒙蔽，之前项目一直可以运行，怎么就报错了呢，日了狗了，于是乎各种百度谷歌的，一顿查，基本上两个答案：

1. servlet-api-xxx.jar包冲突
2. servlet-api-xxx.jar版本问题

各种修改了以后，根本解决不了，看来只能靠自己了，有开始检查pom.xml，目前只能从pom.xml入手了检查了:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <parent>
    <artifactId>dfs-parent</artifactId>
    <groupId>com.gwghk.base.platform</groupId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../parent/pom.xml</relativePath>
  </parent>
  <modelVersion>4.0.0</modelVersion>

  <artifactId>dfs-api</artifactId>
  <version>1.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>${spring.boot.version}</version>
    </dependency>

    ......

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
    <resources>
      <resource>
        <directory>src/main/resources/</directory>
        <filtering>true</filtering>
        <includes>
          <include>**/*.*</include>
        </includes>
      </resource>
    </resources>
  </build>
</project>
```

我家pom是这样的，发现缺少两个属性配置的,没有`<groupId>xxx</groupId>`和`<packaging>jar</packaging>`。  
其实之前都看到这个问题了，因为这项目是我修改别人的代码，就没在乎这个，以为没关系，然后我就加上试试看，是不是这个问题：

```xml
<artifactId>dfs-api</artifactId>
<version>1.0-SNAPSHOT</version>
<groupId>com.gwghk.base.platform</groupId>
<packaging>jar</packaging>
```

再重新编译一次启动试试看，卧槽，竟然成功了，就这个破问题，解决了一天！！！

# 解决办法

问题就出现在pom配置不严谨，该有的还是要有的，不要瞎JB省略。  
在pom中添加`<groupId>xxx</groupId>`和`<packaging>jar</packaging>`就可以正常启动了：

```xml
<artifactId>dfs-api</artifactId>
<version>1.0-SNAPSHOT</version>
<groupId>com.gwghk.base.platform</groupId>
<packaging>jar</packaging>
```