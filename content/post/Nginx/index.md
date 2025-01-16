---
title: Nginx
date: 2025-01-06
slug: nginx
categories:
- Nginx
description: Nginx介绍与常用配置
draft: true
lastmod: 2025-01-06
---

## 优势

1. 提高访问速度
2. 负载均衡
3. 反向代理保护后端

## 配置

在 `conf` 目录下编辑 `nginx.conf` 文件

```text
upstream servers{ # 设置服务集群
	server 192.168.100.128:8080;
	# server 192.168.100.129:8080 weight = 90;
}


server {
    listen       80; # 监听端口
    server_name  localhost;

    location /api/ { # 转发 /api/** 到 /admin/**
	    proxy_pass http://localhost:8080/admin/;
	    proxy_pass http://servers/admin/;  # 使用负载均衡
    }
}
```

### 负载均衡策略

- 轮询：默认
- weight：权重
- ip_hash：根据 IP 分配，每个 IP 固定与一服务器连接
- least_conn：最少连接，优先将请求分配给连接数少的服务器
- url_hash：根据 URL 分配，相同的 URL 将被分配到相同的服务器
- fair：根据响应时间，优先将请求分配给响应时间短的服务器