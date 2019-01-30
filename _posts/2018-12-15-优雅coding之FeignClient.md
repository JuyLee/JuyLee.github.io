---
layout:     post
title:      优雅coding之FeignClient
subtitle:   优雅远程调用
date:       2018-12-15
author:     李俊阳
catalog: true
tags:
    - 优雅coding
    - rpc
    - feignClient
  
---
# 优雅发http请求调用其它服务

##### 一、feignClient介绍

 之前博文，提到过自己写了个面向接口编程，访问其它应用的[博文](https://juylee.github.io/2018/11/29/%E4%BC%98%E9%9B%85coding%E4%B9%8BAPI/)，
 最近在使用springCloud，发现了个神器，FeignClient,这玩意背后的原理，跟我写的那篇博文差不多，但是胜在spring官方提供的，并且提供了其它一些
 场景的使用，我只用到了基于接口的rpc调用，很简单的写法，就能代替之前博文的代码

##### 二、feignClient仅仅代替http请求的调用写法

* pom 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```
* 调用接口代码

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.List;

@FeignClient(name = "demo", url = "${url.demo}")
public interface DemoRpc {

    @RequestMapping(value = "/demo", method = RequestMethod.POST,consumes = MediaType.APPLICATION_JSON_VALUE)
    DemoResp demo(DemoReq demoReq);
}

```
* sprringboot 应用配置
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class DemoApp {

    public static void main(String[] args) {
        SpringApplication.run(DemoApp.class, args);
    }

}

```
* 使用时，直接@AutoWired注解 注入DemoRpc 即可

##### 三、总结
FeignClient还是有很多坑的，特别是与springBoot版本问题，所出现的大坑，使用过程中，找不到原因，建议把feignClient版本降低一点，
这里仅仅是作为一个http请求的替代品进行简单使用，feignClient是支持很对定制的，使用文档建议看官方文档。
    


