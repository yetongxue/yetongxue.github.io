---
title: SpringBoot的redis启动报错：ERR This instance has cluster support disabled
date: 2017-11-28 14:27:23
tags: [Exception,Redis,SpringBoot]
categories: 
  - 异常记录
---
# 异常描述

新建了一个项目，我自己的服务器安装了一个redis，安装的时候，基本上都是默认参数，然后SpringBoot配置如下：

```yml
spring:
  redis:
    cluster:
      nodes: qianxunclub.com:6666
```
<!-- more -->
在项目启动的时候，报错：

```
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'redisTemplate' defined in class path resource [com/qianxunclub/starter/redis/autoconfigure/RedisConfig.class]: Unsatisfied dependency expressed through method 'redisTemplate' parameter 0; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'redisConnectionFactory' defined in class path resource [org/springframework/boot/autoconfigure/data/redis/RedisAutoConfiguration$RedisConnectionConfiguration.class]: Invocation of init method failed; nested exception is redis.clients.jedis.exceptions.JedisDataException: ERR This instance has cluster support disabled
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:749)
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:467)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1138)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066)
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:835)
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:741)
	... 33 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'redisConnectionFactory' defined in class path resource [org/springframework/boot/autoconfigure/data/redis/RedisAutoConfiguration$RedisConnectionConfiguration.class]: Invocation of init method failed; nested exception is redis.clients.jedis.exceptions.JedisDataException: ERR This instance has cluster support disabled
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1628)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:555)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483)
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1138)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066)
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:835)
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:741)
	... 47 common frames omitted
Caused by: redis.clients.jedis.exceptions.JedisDataException: ERR This instance has cluster support disabled
	at redis.clients.jedis.Protocol.processError(Protocol.java:127)
	at redis.clients.jedis.Protocol.process(Protocol.java:161)
	at redis.clients.jedis.Protocol.read(Protocol.java:215)
	at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:340)
	at redis.clients.jedis.Connection.getRawObjectMultiBulkReply(Connection.java:285)
	at redis.clients.jedis.Connection.getObjectMultiBulkReply(Connection.java:291)
	at redis.clients.jedis.Jedis.clusterSlots(Jedis.java:3376)
	at redis.clients.jedis.JedisClusterInfoCache.discoverClusterNodesAndSlots(JedisClusterInfoCache.java:54)
	at redis.clients.jedis.JedisClusterConnectionHandler.initializeSlotsCache(JedisClusterConnectionHandler.java:39)
	at redis.clients.jedis.JedisClusterConnectionHandler.<init>(JedisClusterConnectionHandler.java:17)
	at redis.clients.jedis.JedisSlotBasedConnectionHandler.<init>(JedisSlotBasedConnectionHandler.java:20)
	at redis.clients.jedis.JedisSlotBasedConnectionHandler.<init>(JedisSlotBasedConnectionHandler.java:15)
	at redis.clients.jedis.BinaryJedisCluster.<init>(BinaryJedisCluster.java:41)
	at redis.clients.jedis.JedisCluster.<init>(JedisCluster.java:83)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.createCluster(JedisConnectionFactory.java:306)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.createCluster(JedisConnectionFactory.java:280)
	at org.springframework.data.redis.connection.jedis.JedisConnectionFactory.afterPropertiesSet(JedisConnectionFactory.java:241)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1687)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1624)
	... 58 common frames omitted
Disconnected from the target VM, address: '127.0.0.1:52157', transport: 'socket'

Process finished with exit code 1

```

# 异常分析

从报错误的信息`ERR This instance has cluster support disabled`很明显看得出来，是没有启动redis集群功能，可是我项目配置的集群的配置方式，要么修改代码为单机配置，要么修改redis为集群方式。

# 解决办法

1、可以修改配置为单机redis配置：

```yml
spring:
  redis:
    host: qianxunclub.com
    port: 6666
```

2、在安装redis的目录找到redis配置文件`redis.conf`，里面会找到配置：
```
# cluster-enabled yes
```

把注释去掉就可以了

```
cluster-enabled yes
```