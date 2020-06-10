---
title: Prometheus--（三）Actuator 介绍
date: 2020-06-10 20:19:58
tags:
- Monitor
- Prometheus
categories:
- DevOps
---
## Actuator介绍

### 概览

springboot 实时监控插件 spring-boot-starter-actuator ，提供了很多监控所需的接口，可以对应用系统进行配置查看、相关功能统计等。

其中，actuator 的 endpoints 允许监视应用程序并与之交互.

Spring Boot 包含很多内建的 endpoints ，并且允许自定义节点。
每个独立的 endpoint 都可被设置成  enabled or disabled 和 exposed (允许远程访问) over HTTP or JMX

### 依赖
springboot框架，自带actuator监控，在pom中引入jar包即可使用。
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.0.2.RELEASE</version>
</dependency>
```

### 使用

**1、启动项目后即可访问（默认和服务同端口）**
```
http://localhost:8000/webproject/actuator/health
http://localhost:8000/webproject/actuator/info
```

a. 可修改路由:
在application.propertie 配置文件中添加
```
management.endpoints.web.base-path=/monitor
```

就可以访问
```
http://localhost:8000/webproject/monitor/health
```

b. 可添加endpoint开关：
```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus,restart,pause
management.endpoint.pause.enabled=true
management.endpoint.restart.enabled=true
```

就可以通过如下api管理：
```bash
curl -XPOST http://localhost:8000/actuator/pause
curl -XPOST http://localhost:8000/actuator/restart
```

**2、actuator 提供了很多api**

默认只开放了 health、info两个节点，如果需要公开所有 则在配置文件中继续加入
```properties
management.endpoints.web.exposure.include=*
```

其中 health方法 是不显示具体的内容的，如需要
则继续加入配置
```properties
management.endpoint.health.show-details=always
```

**3、api列表**
```
HTTP方法 路径 描述 鉴权
GET /autoconfig 查看自动配置的使用情况 true
GET /configprops 查看配置属性，包括默认配置 true
GET /beans 查看bean及其关系列表 true
GET /dump 打印线程栈 true
GET /env 查看所有环境变量 true
GET /env/{name} 查看具体变量值 true
GET /health 查看应用健康指标 false
GET /info 查看应用信息
GET /mappings 查看所有url映射 true
GET /metrics 查看应用基本指标 true
GET /metrics/{name} 查看具体指标 true
POST /shutdown 关闭应用（要真正生效，得配置文件开启endpoints.shutdown.enabled: true） true
GET /trace 查看基本追踪信息 true
```

详细可参考：[production-ready](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready)

