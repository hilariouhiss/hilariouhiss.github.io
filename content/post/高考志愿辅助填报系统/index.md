---
title: 高考志愿辅助填报系统
date: 2024-12-04
slug: gaokao-vas
categories:
  - Project
description: 回顾大三软件工程的课程设计，总结一些当时遇到的问题与解决方案
draft: false
lastmod: 2025-03-14
---

## 开发背景

软工课程要求大家组队，并选择老师给出的项目进行开发，我们组一共4人，选择了高考志愿辅助填报系统，分工是两个前端，两个后端。项目要求: 仿造[阳光高考 ]([阳光高考_教育部高校招生阳光工程指定平台](https://gaokao.chsi.com.cn/))网站布局，实现核心功能，至少包括用户管理，院校与专业信息查询，日志功能等三大模块，其中用户管理要求实现0级的 RBAC，日志需要分为系统和用户操作两部分，最后系统需要在本地或云端成功部署。

## 开发

- 使用Spring Boot快速创建 Web 项目，使用 Web、Lombok、Spring Security、Spring Cache、MySQL 和 JPA 依赖。
- 使用三层架构，分为 controller-service- repository 三层。
- 通过自定义 Security 的组件实现用户的权限校验，使用 jjwt 实现的 JWT 进行用户身份信息传输。
	- `JwtAuthenticationFilter` 继承 `OncePerRequestFilter` 实现每个请求只进行一次鉴权
	- `CustomPermissionEvaluator` 实现 `PermissionEvaluator` 接口并被配置，实现提取用户身份信息后进行权限比对
	- `CustomAuthenticationSuccessHandler` 实现 `AuthenticationSuccessHandler` 接口，实现在用户成功登录时生成 JWT 并添加到响应头中
	- `CustomAuthenticationFailureHandler` 则实现用户认证失败的情况，一是用户名或密码错误，二是用户被禁用
- 编写 `CustomGlobalExceptionHandler` 和对应的错误类，如此来实现错误统一处理与返回。
- 使用 Spring Cache 管理缓存，常用注解有 `@Cacheable`、`@CachePut`、`@CacheEvict`、`@Caching` 和类注解 `@CacheConfig`
- 日志功能使用 Log4j2，编写 xml 实现系统日志和用户操作日志输出到不同位置，并以不同的时间和空间留存。
- 部署使用 docker，nginx，tomcat。

## 困难与解决

1. 项目开发初期，由于接口设计与需求分析大家讨论不彻底，导致开发途中不断被要求增加新功能，尤其是在院校专业查询板块，时常有不同的查询条件被要求添加。而在后期系统合一进行部署时，许多接口前后端不统一，导致时常需要进行小修改，经常我确定后端功能是对的，他也确定前端是对的，最后排查时发现只是接口名字或发送的 JSON 格式不对。
	**解决**: 加强前后端沟通，尤其是接口方面，不能口头上简单地说一下，一定要给出详细的文档，否则很容易双方各有各的理解，导致后期对接麻烦。
1. 在定制 Security 的权限校验功能时，网上的资料大多是老版本的 Security 的 config 如何配置，而我使用的 Security 较新，所以耗费了大量的时间去查找新 Security 的配置方法。并且这一问题也发生在使用 jjwt 开发 JWT 时，大多资料不够新，即使是问 ChatGPT 也无法得到正确答案。
	**解决**: 最考验信息检索整合能力的一集，搜索时添加时间限制，只看最近一两年甚至几个月的，最好参照官方文档进行使用。使用 jjwt 时，我就是通过网上的资料了解到如何将其加入 Spring 中，再参考官方文档使用新的方法将旧的替换。
3. 系统整合部署时问题频发，首先是跨域和拒绝访问，在改动 Security 的配置后时常出现，每次都是不同的原因造成。第二是使用 Tomcat 启动不成功，最后发现是 Tomcat 的 web.xml 配置需要更改。第三是部署后由于系统启动时需要读取三张 Excel 表格，同时录入三四千条数据，导致 docker 启动经常卡住，并且由于容器启动顺序和 Spring 启动问题导致录入数据工作不正常。
	**解决**: 改动配置时要注意是否会增加访问限制，请求源，请求方法和请求头等设置都有可能造成拒绝访问。设置 docker 容器启动顺序和条件，并且设置 ApplicationReadyListener，当 App 完全启动时再多线程地读入数据。

## 缺点

由于开发经验不足，在前后端数据传输部分开发地较简单，直接在实体类的各个属性上，使用了大量的序列化控制，通过这种方式来控制只将必须的数据返回给前端。
项目答辩通过一段时间才发现可通过创建 Post Body，VO 和 DTO 之类的对象来控制，并且此种方式还能使项目框架更加分明，控制数据传输更容易。

为了便利性而决定使用JPA，若是需要提高系统性能和更好控制 SQ L语句，应该使用 mybatis 等 ORM 框架。在开发后期，仍然通过 `@Query` 编写了一部分原生的 SQL 语句。

此项目开发期间，多数问题是由于个人过于追求新版本而造成，例如 Security 的配置问题，在如何按照最新的格式去编写方面，花费了大量的时间，反而知道了用法后，具体的配置只消耗少许时间。

## 项目地址

[hilariouhiss/gaokao-vas: 高考志愿辅助填报系统](https://github.com/hilariouhiss/gaokao-vas)