---
title: "PHP非阻塞的几种实现方式"
slug: "php-several-ways-non-blocking"
description: "为让 PHP 在后端处理长时间任务时不阻塞，快速响应页面请求，部分业务后台继续执行，可以有以下措施："
date: "2022-01-27T07:42:03Z"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "php"
  - "非阻塞"
---

为让 PHP 在后端处理长时间任务时不阻塞，快速响应页面请求，部分业务后台继续执行，可以有以下措施：

### 使用 fastcgi_finish_request()

如果 PHP 与 Web 服务器使用了 PHP-FPM（FastCGI 进程管理器），那通过 fastcgi_finish_request() 函数能马上结束会话，而 PHP 线程可以继续在后台运行。

官方对此函数的解释：

    此函数冲刷(flush)所有响应的数据给客户端并结束请求。
    这使得客户端结束连接后，需要大量时间运行的任务能够继续运行。

示例代码：

```php
echo "program start...";

file_put_contents('log.txt','start-time:'.date('Y-m-d H:i:s'), FILE_APPEND);

fastcgi_finish_request();
// 到这里http响应就结束了

// 以下代码会在后台继续执行
sleep(1);
echo 'debug...';
file_put_contents('log.txt', 'start-proceed:'.date('Y-m-d H:i:s'), FILE_APPEND);

sleep(10);
file_put_contents('log.txt', 'end-time:'.date('Y-m-d H:i:s'), FILE_APPEND);
```

从输出结果可看到，页面打印完 `program start...`，输出第一行到 log.txt 后会话就返回了，所以后面的 `debug...` 不会在浏览器上显示，而 log.txt 文件能完整地接收到三个完成时间。

### 使用 fsockopen()

使用 fsockopen() 打开一个网络连接或者一个Unix套接字连接，再用 stream_set_blocking() 非阻塞模式请求：

```php
$fp = fsockopen("www.example.com", 80, $errno, $errstr, 30);

if (!$fp) {
    die('error fsockopen');
}

// 转换到非阻塞模式
stream_set_blocking($fp, 0);

$http = "GET /save.php  / HTTP/1.1\r\n";
$http .= "Host: www.example.com\r\n";
$http .= "Connection: Close\r\n\r\n";

fwrite($fp, $http);
fclose($fp);
```

### 使用 cURL

利用cURL中的 curl_multi_* 函数发送异步请求

```php
$mh = curl_multi_init();
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "http://localhost/");
curl_multi_add_handle($mh, $ch);
curl_multi_exec($mh, $active);

curl_close($ch);
curl_multi_remove_handle($mh, $ch);
curl_multi_close($mh);

echo "End\n";
```

### 使用 pcntl_fork()
安装 `pcntl` 扩展，使用 `pcntl_fork()` 生成子进程异步执行任务，个人觉得是最方便的，但是缺点是容易出现僵尸进程。

参考文章：[https://www.jianshu.com/p/de0b74f58f50](https://www.jianshu.com/p/de0b74f58f50)

```php
$pid = pcntl_fork()
if ($pid == 0) {
    child_func();    //子进程函数，主进程运行
} else {
    father_func();   //主进程函数
}

echo "Process " . getmypid() . " get to the end.\n";
 
function father_func() {
    echo "Father pid is " . getmypid() . "\n";
}

function child_func() {
    sleep(6);
    echo "Child process exit pid is " . getmypid() . "\n";
    exit(0);
}
```

### PHP原生支持

介绍文章：[https://www.laruence.com/2015/05/28/3038.html](在PHP中使用协程实现多任务调度)

### 使用 Gearman/Swoole 扩展

Gearman 是一个具有 php 扩展的分布式异步处理框架，能处理大批量异步任务。

Swoole 最近很火，有很多异步方法，使用简单。

    既然都用Swoole了，为什么不考虑一下Go呢？

### 使用缓存和队列

使用redis等缓存、队列，将数据写入缓存，使用后台计划任务实现数据异步处理。（这个方法在常见的大流量架构中应该很常见吧。）

### 调用系统命令

极端的情况下，可以调用系统命令，可以将数据传给后台任务执行，个人感觉不是很高效。

```php
$cmd = 'nohup php ./processd.php $someVar >/dev/null  &';
`$cmd`
```

参考文章：[https://www.awaimai.com/660.html](https://www.awaimai.com/660.html)


