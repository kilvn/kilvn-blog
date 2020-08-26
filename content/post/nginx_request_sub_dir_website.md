---
title: "访问 Nginx 二级目录中的 Laravel 项目"
slug: "nginx_request_sub_dir_website"
description: "laravel项目放在 aaa 下，目的是需要通过局域网IP访问，如果不做转发，可以访问首页，但是子页面无法访问，因为没有配置rewrite。"
date: "2019-07-19T17:54:24+08:00"
thumbnail: ""
categories:
  - "Develop"
tags:
  - "nginx"
  - "laravel"
---
这个问题折腾了挺久时间都没成功，尝试了定义root path，也尝试了proxy_pass都不行。直到看到一篇文章，介绍的是目录伪静态转发才搞定……

根目录为 /，laravel项目放在 aaa 下，目的是需要通过局域网IP访问laravel项目，如果不做转发，可以访问首页，但是子页面无法访问，因为没有配置rewrite。

以下是转发代码：

```
location /aaa/public/ {
    if (!-e $request_filename){
        rewrite ^/aaa/public/(.*)$ /aaa/public/index.php?s=$1 last;
    }
}
```

参考：[linux 下 nginx 二级目录中访问Laravel项目](https://blog.csdn.net/qdslp/article/details/89010834)
