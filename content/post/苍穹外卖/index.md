---
title: Spring Boot + SSM
date: 2025-01-06
slug: sky-takeaway
categories:
- 开发
description: 初步学习企业级项目开发
draft: true
lastmod: 2025-01-06
---

## 软件开发整体流程

1. 需求分析：需求规格说明书、产品原型
2. 设计：UI设计、数据库设计、接口设计
3. 编码：项目代码、单元测试
4. 测试：测试用例、测试报告
5. 上线运维：软件安装与配置

## 项目介绍

### 定位

面向餐饮企业及其用户的一款软件

### 功能模块

#### 管理端

员工管理、分工管理、菜品管理、套餐管理、订单管理、工作台、数据统计、来单提醒

#### 用户端

微信登录、商品浏览、购物车、用户下单、微信支付、历史订单、地址管理、用户催单

### 技术选型

用户层：node、vue、elementUI、微信小程序、echarts

网关层：nginx

应用层：Spring Boot、Spring Task、httpclient、Spring Cache、JWT、阿里云OSS、Knife4j、POI、WebSocket

数据层：MySQL、redis、mybatis、pagehelper、spring data redis

工具：git、maven、Junit、Bruno

