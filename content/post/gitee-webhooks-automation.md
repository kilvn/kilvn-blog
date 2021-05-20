---
title: "Gitee WebHooks 自动化部署"
slug: "gitee-webhooks-automation"
description: "首先创建供gitee请求的php脚本，然后在gitee的webhooks做配置，接着配置网站运行目录权限，最后配置`www-data`用户的请求密钥，并给目录赋予`www-data`用户组。"
date: "2021-05-20T15:57:09+08:00"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "gitee"
  - "webhooks"
---

1. 首先创建供gitee请求的php脚本

`depoy.php`

```php
<?php

class webhook
{
    /**
     * @var self
     */
    protected static self $instance;

    // webhook上设置的secret
    protected static string $secret = '你的webhooks密码';
    protected string $localPath;
    protected string $gitRemote;
    protected string $logPath;
    protected string $branch;
    protected string $shell;

    /**
     * webhook constructor.
     */
    public function __construct()
    {
        try {
            set_time_limit(0);
            require 'vendor/autoload.php';
            $dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
            $dotenv->load();

            $this->branch = 'xx_dev';
            $this->localPath = realpath(__DIR__);
            $this->gitRemote = 'git@gitee.com:kilvn/fth.git';
            $this->logPath = $_ENV['LOG_ROOT'] . '/git-webhook.log';
            $this->shell = 'cd ' . $this->localPath . ' && git pull';

            $this->index();
        } catch (Exception $exception) {
            echo $exception->getMessage();
        }
    }

    protected function index()
    {
        $requestBody = file_get_contents('php://input');
        if (empty($requestBody)) {
            header("HTTP/1.1 404 Not Found");
            header("Status: 404 Not Found");
            die('send fail');
        }

        //解析Git服务器通知过来的JSON信息
        $content = json_decode($requestBody, true);
        if ($content['password'] != self::$secret) {
            header("HTTP/1.1 404 Not Found");
            header("Status: 404 Not Found");
            exit('deny');
        }

        $branch = str_replace('refs/heads/', '', $content['ref']);

        //若是目标分支且提交数大于0
        if ($branch == $this->branch && $content['total_commits_count'] > 0) {
            $res = PHP_EOL . 'pull start --------' . PHP_EOL;
            $do = self::doShell($this->shell);
            $do = json_encode($do);
            $res .= $do;
            $res_log = '-------------------------' . PHP_EOL;
            $res_log .= $content['user_name'] . ' 在' . date('Y-m-d H:i:s') . '向' . $content['repository']['name'] . '项目的' . $content['ref'] . '分支push了' . $content['total_commits_count'] . '个commit：';
            $res_log .= $res . PHP_EOL;
            $res_log .= 'pull end --------' . PHP_EOL;
            $this->logs($res_log);

            exit($do);
        }
    }

    /**
     * 执行shell命令
     *
     * @param $cmd
     * @param null $cwd
     * @return array|string
     */
    protected static function doShell($cmd, $cwd = null)
    {
        $descriptorspec = array(
            0 => array('pipe', 'r'), // stdin
            1 => array('pipe', 'w'), // stdout
            2 => array('pipe', 'w'), // stderr
        );

        $proc = proc_open($cmd, $descriptorspec, $pipes, $cwd, null);

        // $proc为false，表明命令执行失败
        if ($proc == false) {
            return '命令执行出错！';
        } else {
            $stdout = stream_get_contents($pipes[1]);
            fclose($pipes[1]);
            $stderr = stream_get_contents($pipes[2]);
            fclose($pipes[2]);
            $status = proc_close($proc); // 释放proc
        }

        return array(
            'stdout' => $stdout, // 标准输出
            'stderr' => $stderr, // 错误输出
            'retval' => $status, // 返回值
        );
    }

    /**
     * 写入日志到log文件中
     *
     * @param $res
     */
    protected function logs($res)
    {
        file_put_contents($this->logPath, $res, FILE_APPEND);
    }
}

try {
    new webhook;
} catch (Exception $exception) {
    echo $exception->getMessage();
}
```

这个脚本使用的是简单的明文密码，没有使用签名密钥。

`xx_dev`需要修改为当前项目需要更新的git分支，`你的webhooks密码`根据自己情况修改。

#### 2. 在gitee的webhooks做配置

#### 3. 配置网站运行目录权限

通过查看`php-fpm.conf`配置文件，我的是使用 `www-data`用户运行php-fpm的。

所以给项目目录`www-data`用户组权限，我的项目运行目录是 `/www/wwwroot/blog`：

```shell
chown -R www-data:www-data /www/wwwroot/blog
```

#### 4. 配置`www-data`用户的请求密钥，并给目录赋予`www-data`用户组

我的服务器是ubuntu20，`www-data`用户的ssh目录在`/var/www/.ssh`

```shell
mkdir /var/www/.ssh && cp -r ~/.ssh/* /var/www/.ssh/ && chown -R www-data:www-data /var/www/.ssh/
```

然后去gitee测试下吧。
