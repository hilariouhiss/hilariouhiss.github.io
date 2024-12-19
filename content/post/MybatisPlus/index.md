---
title: MybatisPlus
date: 2024-12-16 19:56:06
slug: mybatisplus
categories:
  - 知识
  - MybatisPlus
summary: 学习 MybatisPlus 的记录
draft: true
---

## 介绍



## 使用方法

### 引入依赖

Spring Boot 3版本如下，boot~~3~~ 即为 2 版本

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.9</version>
</dependency>
```

### 配置

在 `application.yml` 文件中添加数据源配置，示例如下：

```yaml
# DataSource Config
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://${MYSQL_ADDRESS:localhost:3306}/${DATABASE}?serverTimezone=Asia/Shanghai
    username: username
    password: password
```

MySQL的 `url` 串常使用配置：`useSSL=false&serverTimezone=UTC&useUnicode=true&characterEncoding=utf8`

禁用 SSL，设置时区为 UTC，使用 Unicode 字符集和 UTF-8 编码

## 常用注解
