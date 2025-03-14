---
title: Spring Security 6.0 的实现原理
date: 2024-12-25
slug: spring-security-6-principles
categories:
  - Spring
description: 彻底搞懂 Spring Security 6.0 的实现原理
draft: true
lastmod: 2025-03-14
---
参考来源：[一文带你读懂Spring Security 6.0的实现原理本文带你深入理解Spring Security的实现原理和基 - 掘金](https://juejin.cn/post/7260000714788896828)

## 认证与鉴权

认证（Authentication）与鉴权（Authorization）

认证的重点是“你是谁？”，鉴权的重点是“你能干什么？”

认证常用 username - password、生物识别、双重验证或多重验证等方式进行，目的时确保只有合法的用户才能访问系统。

鉴权位于认证之后，是对

## 重点关注

- 总体架构
- 身份认证
- 鉴权控制

## 基本思路

在请求入口处进行安全相关的校验和处理，好处有几方面：
1. 集中管理，简化代码：安全规则统一管理，无需在每个controller中写一遍，减少了重复代码，降低了出错的可能性，更方便了后续可能的修改。
2. 提升性能，增强安全性：当发现一个请求是恶意的时候，可以直接拦截，而不会解析到对应的controller，从而节约系统资源和提升性能。
3. 一致性与可扩展性：可让所有请求应用相同的安全规则，使得系统响应更加可预测；分布式架构下，当有大流量时，可水平扩展网关服务来处理。

JavaEE 规范中，Filter 和 Servlet 均满足这个要求，