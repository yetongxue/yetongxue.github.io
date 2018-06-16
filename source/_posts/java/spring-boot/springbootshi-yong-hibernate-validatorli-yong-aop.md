---
title: SpringBoot使用hibernate-validator利用AOP实现统一参数校验
date: 2018-04-26 02:22:12
tags: [Java,SpringBoot]
categories: 
  - Java
  - SpringBoot
---

# 引入maven包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.5.9.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
    <version>2.0.1.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>3.0.1-b08</version>
</dependency>
```
<!--more-->
# 注入Validator的bean
```java
package com.qianxunclub.starter.web.autoconfigure;

import lombok.extern.slf4j.Slf4j;
import org.hibernate.validator.HibernateValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;

/**
 * @author chihiro.zhang
 */
@Slf4j
@Configuration
public class ValidatorConfiguration {

    @Bean
    public Validator getValidatorFactory(){
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class).configure().failFast(true).buildValidatorFactory();
        return validatorFactory.getValidator();
    }
}

```

# 添加参数校验的AOP

```java
package com.qianxunclub.starter.web.validator;

import com.qianxunclub.common.framework.constant.CommonReturnCode;
import com.qianxunclub.common.framework.response.Result;
import com.qianxunclub.utils.HttpResponseUtil;
import com.qianxunclub.utils.JsonUtil;
import lombok.AllArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletResponse;
import javax.validation.ConstraintViolation;
import javax.validation.Path;
import javax.validation.Validator;
import java.lang.reflect.Method;
import java.util.Set;

/**
 * @author chihiro.zhang
 */
@Slf4j
@Aspect
@Component
@AllArgsConstructor
public class ValidatorAspect {

    private Validator validator;

      //定义校验的包位置
      @Pointcut("execution(* com.qianxunclub..*.web.*.*(..))")
      public void pointcut() {
      }

    /**
     * 入参校验
     * @param joinPoint
     * @throws Throwable
     */
    @Around("pointcut()")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        Result result = Result.newBuilder().build();
        String methodName = "";
        try{
            Signature sig = joinPoint.getSignature();
            MethodSignature msig = (MethodSignature) sig;
            Object target = joinPoint.getTarget();
            Method currentMethod = target.getClass().getMethod(msig.getName(), msig.getParameterTypes());
            methodName = currentMethod.getName();
        } catch (Exception e){
            log.debug("无法获取方法名称" ,e);
        }
        Object[] args = joinPoint.getArgs();
        for (Object arg : args){
            log.debug("→→→→→" + methodName + ">>>>Into parameter :" + JsonUtil.objectToJson(arg));
            if(arg != null){
                Set<ConstraintViolation<Object>> constraintViolations = validator.validate(arg);
                if(constraintViolations.size() > 0){
                    for (ConstraintViolation<Object> constraintViolation : constraintViolations) {
                        Path property = constraintViolation.getPropertyPath();
                        String name = property.iterator().next().getName();
                        result = Result.newBuilder().fail().code(CommonReturnCode.PARAM_ERROR).message("[" + name + "]" + constraintViolation.getMessage()).build();
                        break;
                    }
                    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                    HttpServletResponse response = attributes.getResponse();
                    String responseStr = JsonUtil.objectToJson(result);
                    log.error(responseStr);
                    log.debug("→→→→→" + methodName + ">>>>Return to the result :" + responseStr);
                    HttpResponseUtil.setResponseJsonBody(response,responseStr);
                    return null;
                }
            }
        }
        Object object = joinPoint.proceed();
        log.debug("→→→→→" + methodName + ">>>>Return to the result :" + JsonUtil.objectToJson(object));
        return object;
    }
}

```

上面`@Pointcut("execution(* com.gwghk..*.web.*.*(..))")为Controller`切点位置
BaseResponse为一个统一返回对象，这个可以自定义

# 使用
使用是需要再Controller入参对象添加校验注解即可：
```java
@Data
public class RegisteProxyParam {
    @NotEmpty
    private String appId;
}
```

