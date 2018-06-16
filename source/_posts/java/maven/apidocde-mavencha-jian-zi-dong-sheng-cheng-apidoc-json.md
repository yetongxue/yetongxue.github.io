---
title: apidoc的maven插件，自动生成apidoc.json
date: 2018-05-16 02:41:37
tags: [Java,Maven]
categories: 
  - Java
  - Maven
---


插件是用apidoc插件生成文档的，具体使用方式可查看官网：http://apidocjs.com/  
该插件不会直接生成APIDOC文档，只会自动生成apidoc.json文件，需要执行`apidoc`命令才可以生成  
apidoc.json文件会生成在项目根目录apidoc文件夹下
<!--more-->
# 下载依赖包
可配置MAVEN仓库`https://oss.sonatype.org/content/groups/public`
或者
下载源码包进行编译打包:https://gitee.com/qianxunclub/qianxunclub-maven-plugin
```
git clone https://gitee.com/qianxunclub/qianxunclub-maven-plugin.git
```
```
cd qianxunclub-maven-plugin
```
```
mvn clean install
```


# 编辑pom.xml，引入maven plugin
在项目的pom文件中引入以下：
```
<plugin>
	<groupId>com.qianxunclub</groupId>
	<artifactId>qianxunclub-plugin-apidoc</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<executions>
		<execution>
			<goals>
				<goal>apidoc</goal>
			</goals>
		</execution>
	</executions>
</plugin>
```

在`properties`定义API的生成规范：
```
<properties>
	<apidoc.skip>false/</apidoc.skip>
	<apidoc.gen>false</apidoc.gen>
	<apidoc.url>http://ip:port/</apidoc.url>
	<apidoc.sampleUrl>http://ip:port/</apidoc.sampleUrl>
</properties>
```
apidoc.skip：编译代码是否跳过生成apidoc.json  
apidoc.gen：是否覆盖更新apidoc.json  
apidoc.url：实例接口前缀  
apidoc.sampleUrl：生成测试方法的请求地址  

# 开始生成
执行命令：
```
mvn clean package
```
可以添加以下参数：
```
mvn clean package -Dapidoc.skip=true
```

-Dapidoc.skip=true：编译代码是否跳过生成apidoc.json  
-Dapidoc.gen=true：是否覆盖更新apidoc.json  
-Dapidoc.url=xxx：实例接口前缀  
-Dapidoc.sampleUrl=xxx：生成测试方法的请求地址  

如果出现以下字样，说明生成完成：
```
apidoc.json完成
```

# 生成api文档
在项目跟目录执行：
```
apidoc -i apidoc/ -o API文档存放目录/
```

打开API文档存放目录中的`index.html`即可查看文档。