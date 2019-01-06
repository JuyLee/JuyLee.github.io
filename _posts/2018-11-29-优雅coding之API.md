---
layout:     post
title:      优雅coding之API
subtitle:   使用注解和动态代理实现面向接口编程
date:       2018-11-29
author:     李俊阳
catalog: true
tags:
    - 优雅coding
    - 注解
    - 动态代理
    - 面向接口编程
  
---
# 优雅coding之API

##### 一、优雅coding实践

最近，做了个分布式调度平台的项目，调研后决定基于xxl-job封装一个定时任务框架，xxl-job因为只依赖mysql，架构简单，所以选择使用，
coding过程中，写了个小框架，针对xxl-job controller层的调用，觉得代码风格比较优雅，所以写了这篇博文，这个思路其实可以用到我们
调用任何的第三方系统。


##### 二、思路分享

* 调用第三方过程，跟我们写controller其实一样，所以有了把调用第三方的API，最后达成的效果与spring的controller效果一致
* url封装 spring有@RequestMapping，所以自己自定义了一个类似的注解@URLMapping
* 面向接口编程 类似MyBatis的mapper层，属于面向接口编程，其实底层都是通过动态代理，调用sqlSessionTemplate,操作数据库，我的底层无非就是
发送http请求，封装参数，url，返回值无非是使用json转换为强类型，空值方法做特殊处理

##### 三、代码部分

* 自定义注解部分

```java

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface UrlMapping {

    String value() default "";


}
```
* 动态代理部分

```java

import com.alibaba.fastjson.JSON;
import com.saic.quartzbot.api.info.ReturnT;
import com.saic.quartzbot.exception.EngineServiceException;
import com.saic.quartzbot.util.RestTemplateUtil;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;

@Component
public class ApiHandler implements InvocationHandler {

    @Value("${xxl.admin.url}")
    private String URL_XXL_JOB_ADMIN;


    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        UrlMapping urlMapping = method.getAnnotation(UrlMapping.class);
        RestTemplate restTemplate = RestTemplateUtil.getRestTemplate();
        //关键代码就这一句 发送请求，封装参数
        ResponseEntity<String> responseEntity = restTemplate.postForEntity(URL_XXL_JOB_ADMIN + urlMapping.value(), RestTemplateUtil.buildReqWithCookie(args != null ? args[0] : args), String.class);
        //处理返回值
            ReturnT returnMsg = (ReturnT) JSON.parseObject(responseEntity.getBody(), method.getReturnType());
            if (returnMsg.getCode() != ReturnT.SUCCESS_CODE) {
                throw new EngineServiceException(returnMsg.getCode(), returnMsg.getMsg());
            }
            return returnMsg;
    }

}
```

* 生成代理,并交由spring管理

```java
import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.Proxy;

@Component("xxlApi")
public class ApiFactory implements FactoryBean<XxlApi> {

    @Autowired
    private ApiHandler apiHandler;
    
    //生成代理
    @Override
    public XxlApi getObject() throws Exception {
        return (XxlApi) Proxy.newProxyInstance(this.getClass().getClassLoader(), new Class[]{XxlApi.class}, apiHandler);

    }

    @Override
    public Class<?> getObjectType() {
        return XxlApi.class;
    }
}
```

* API接口部分

```java

import java.util.List;
import java.util.Map;

public interface XxlApi {

    @UrlMapping("/jobinfo/add")
    ReturnT<Integer> add(XxlJobInfo xxlJobInfo);

    @UrlMapping("/jobinfo/update")
    ReturnT<String> update(XxlJobInfo xxlJobInfo);

    @UrlMapping("/jobinfo/remove")
    ReturnT<String> remove(JobIdReq jobIdReq);

    @UrlMapping("/jobinfo/pause")
    ReturnT<String> pause(JobIdReq jobIdReq);

    @UrlMapping("/jobinfo/resume")
    ReturnT<String> resume(JobIdReq jobIdReq);

    @UrlMapping("/jobinfo/trigger")
    ReturnT<String> triggerJob(JobIdReq jobIdReq);

}
```
可以看到，最后的API层实现了类似controller层的代码风格，又兼具面向接口编程的特点，整体代码风格比较优雅，如果需要增加新的接口，
只需要定义好入参和返回，因为xxl-job的接口基本都是post，所以代码里面并没有做get请求和返回值是void的处理，理清思路，随手就加上去了。

##### 四、总结
这里只是比较简单的搭建了一个面向接口编程的例子，笔者其实在很久前，阅读mybatis的源码，发现了很多优雅的coding的实现方式，也早已经在
实践了。阅读源码不仅仅能加深我们对框架的理解，还能提高我们掌握优雅coding的技巧，其实原理很简单，也就用到了自定义注解和动态代理部分，

    


