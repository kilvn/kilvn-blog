---
title: "ubuntu 安装 tengine 和 php-fpm 运行php脚本"
slug: "ubuntu-install-tengine-and-php-fpm"
description: "Tengine是由淘宝网发起的Web服务器项目。它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如淘宝网，天猫商城等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。"
date: "2019-04-24T19:12:32+08:00"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "ubuntu"
  - "tengine"
  - "php-fpm"
---

Tengine是由淘宝网发起的Web服务器项目。它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性。Tengine的性能和稳定性已经在大型的网站如淘宝网，天猫商城等得到了很好的检验。它的最终目标是打造一个高效、稳定、安全、易用的Web平台。

从2011年12月开始，Tengine成为一个开源项目，Tengine团队在积极地开发和维护着它。Tengine团队的核心成员来自于淘宝、搜狗等互联网企业。Tengine是社区合作的成果，我们欢迎大家参与其中，贡献自己的力量。

Tengine完全兼容Nginx，因此可以参照Nginx的方式来配置Tengine。

Tengine官方下载地址：[http://tengine.taobao.org/download_cn.html](http://tengine.taobao.org/download_cn.html)

目前最新版本为 tengine-2.3.3。

### tengine 编译安装

```
wget http://tengine.taobao.org/download/tengine-2.3.3.tar.gz
tar -zxvf tengine-2.3.3.tar.gz
cd tengine-2.3.3
apt update

# 安装gcc编译器 make，众多依赖包
apt install -y gcc g++ make libpcre3 libpcre3-dev openssl libssl-dev zlib1g-dev

# --sbin 指定 tengine 的安装位置为 /usr/sbin/nginx，这样就可以直接指向 nginx [xxx] 指令
./configure --prefix=/etc/nginx/ --sbin-path=/usr/sbin/nginx --with-stream && make && make install

# 安装完成后查看 tengine 版本
nginx -v
```

### 安装php

如果你的服务器没有php7.4，比如apt仓库只到php7.2，那么添加这个源（必须以root运行）[参考链接](https://launchpad.net/~ondrej/+archive/ubuntu/php)

```
sudo add-apt-repository ppa:ondrej/php
```
##### 安装php7.4

```
apt install -y php7.4-fpm

# 安装完成之后查看php版本
php -v

# 查看php已安装的扩展
php -m
```

到这里还没结束，还要对 nginx(tengine) 和 php 进行配置，才能运行php脚本。

##### 修改PHP的PID监听为端口监听
```
vim /etc/php/7.4/fpm/pool.d/www.conf

// 大约36行，加上 ; 注释掉
listen = /run/php/php7.4-fpm.sock

// 紧接着下面一行添加 9000 监听端口
listen = 127.0.0.1:9000

// 重启php
php-fpm7.4
```

##### 修改nginx配置，以支持php解析

```
vim /etc/nginx/conf/nginx.conf

// 将 server{} 中的 (location /) 和 (location ~ \.php$) 取消注释，并在 (location /) 中 index后添加index.php以指定默认php文件首页，修改后的配置像下面这样

server{
……我是省略号……
        location / {
            root   html;
            index index.php index.html index.htm;
        }
……我是省略号……
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }
……我是省略号……
}

// 平滑重启nginx
nginx -s reload

// 如果nginx重启报[error] open()错误，则需要执行两步：

vim /etc/nginx/conf/nginx.conf

// 将开头 pid 那行取消注释
pid         logs/nginx.pid

// 然后生成 nginx.pid
nginx -c /etc/nginx/conf/nginx.conf
```

最后，在 /etc/nginx/html/ 目录创建php文件测试一下子吧。

##### 配置vhosts虚拟主机

创建vhosts配置目录

```shell
mkdir /etc/nginx/conf/vhosts
```

编辑nginx配置文件，引入vhosts下的虚拟主机配置文件

```shell
vim /etc/nginx/conf/nginx.conf
```

在http组第一个server后面引入配置文件

```
include vhosts/*.conf;
```

然后在vhosts目录创建网站配置信息

```
cd vhosts
touch xxx.com.conf
vim xxx.com.conf
```

配置文件的内容如下

```
server {
    
    listen 80;
    listen [::]:80;
    
    # For https
    # listen 443 ssl;
    # listen [::]:443 ssl ipv6only=on;
    # ssl_certificate /etc/nginx/ssl/default.crt;
    # ssl_certificate_key /etc/nginx/ssl/default.key;
    
    disable_symlinks off;
    
    charset utf-8;
    server_name xxx.com;
    
    location / {
        root /www/wwwroot/xxx.com/public;
        index index.php index.html index.htm;
        
        if (!-e $request_filename) {
            rewrite  ^(.*)$  /index.php?s=/$1 last;
        }
    }
    
    location ~ \.php$ {
        root /www/wwwroot/xxx.com/public;
        fastcgi_pass  127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_buffers 16 16k;
        fastcgi_buffer_size 32k;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        #fixes timeouts
        fastcgi_read_timeout 600;
        include fastcgi_params;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

然后重启nginx，用域名打开网站看看吧（前提是已解析好域名）。

如果想用公网IP打开网站，server_name 后面下划线就行。

```
server_name _;
```

### tengine 完美卸载

```
apt remove nginx nginx-common && apt purge nginx nginx-common && apt autoremove && apt remove nginx-full nginx-common
```

参考文章:[tengine配置虚拟主机的三种方式](https://blog.csdn.net/hll19950830/article/details/79751482)
