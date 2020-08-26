---
title: "Nginx反向代理远程MySQL"
slug: "nginx-proxy-mysql"
description: ""
date: "2020-03-03T20:08:57+08:00"
thumbnail: ""
categories:
  - "Develop"
tags:
  - "nginx"
  - "mysql"
  - "反向代理"
---
##### 背景

情况是这样的，我们一个项目的客户是AWS的RDS，因为业务需要，我们要连接客户的RDS，但是客户那边只能添加一台我们机器的IP白名单。

所以我们在AWS这台RDS的同区域买了一台EC2云服务器，然后提供公网IP，这样RDS那边就添加了我们的IP白名单，可以在这台EC2上面连接那台RDS了。。。

但是ubuntu命令行操作数据库始终不太方便啊，比如：查询结果导出为Excel，导出来的文件表头是没有问题，每一条数据也都按行区分，但是多列合并放到第一列里面了，这样数据就没发操作和查看了。。。

所以打算用nginx的反向代理试一下。

##### 准备工作

使用的是nginx的stream模块，测试前需要查看nginx是否在编译时开启了stream模块：

```shell
nginx -m
```

如果看到 ngx_stream_module 字样就是已经编译了stream模块（我用的是Tengine）。

##### 反向代理配置

编辑nginx配置文件：

```shell
vim /etc/nginx/conf/nginx.conf
```

注意 stream 位于配置文件顶层，和 http 同级。

```
# nginx 代理
stream {
    # 设置代理超时时间，设的大一些，避免长连接因为超时时间而中断
    proxy_timeout 3d;
    
    server {
        listen 3307;
        listen [::]:3307;
        
        proxy_pass xxx.com:3306;
    }
}
```
重启nginx

```shell
# 使修改后的配置文件生效
nginx -c /etc/nginx/conf/nginx.conf
# 平滑重启nginx
nginx -s reload
```

##### 测试

客户端连接你的反代机器IP:3307，实际上相当于连接到了xxx.com:3306这个数据库。

如果使用了云服务器，注意在策略组放行 tcp 3307端口。

