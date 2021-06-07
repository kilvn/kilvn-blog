---
title: "Nginx net::ERR_HTTP2_PROTOCOL_ERROR 200 错误"
slug: "nginx-net-rrr_http2_protocol_error-200"
description: "在nginx反向代理location字段加入以下内容并重启nginx服务即可解决。proxy_max_temp_file_size 0;"
date: "2021-06-07T20:58:33+08:00"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "nginx"
  - "ERR_HTTP2_PROTOCOL_ERROR"
---

**错误描述：** 网站突然在Chrome浏览器或者Chrome内核的浏览器下打不开了，按F12调试发现部分资源报错net::ERR_HTTP2_PROTOCOL_ERROR，状态码又是200，表示客户端到服务器的链接是正常的，能建立正常链接。

初始怀疑硬件防火墙问题，因为之前访问正常，突然访问错误，但是查询硬件防火墙并未发现任何内容拦截记录。因为是通过nginx反向代理后端nginx服务器，所以首先排除后端服务器的问题。

因为是从内网穿透到公网涉及到安全问题，所以前后端服务器均采用的是https链接，并启用了http2,但是直接访问后端服务器正常，浏览器没有报任何错误，故推测错误原因应该来源于前端反代nginx服务器上。

但是此时发现个奇怪的现象，用firefox浏览器访问正常，那这个问题就应该是在Chrome浏览器浏览器上下手了。

查了半天资料发现有一个解决方案，在nginx反向代理location字段加入以下内容并重启nginx服务即可解决。

```nginx
proxy_max_temp_file_size 0;
```

加入上面内容后重启nginx服务后Chrome浏览器访问正常。

原文来自：[nginx net::ERR_HTTP2_PROTOCOL_ERROR 200 错误 | 一只小桃桃](https://yisca.cn/2310.html)


