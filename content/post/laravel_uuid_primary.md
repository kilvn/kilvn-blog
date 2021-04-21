---
title: "Laravel 使用 UUID 作为用户表主键并使用自定义用户表字段"
slug: "laravel_uuid_primary"
description: "最近在用 Laravel 5.6 做一个项目，涉及到用户表的自定义字段和 UUID 作为主键，各种 Google 花了我很长时间，所以本篇文章用来记录一下实现思路，以防后人踩坑。"
date: "2021-01-25T03:08:36Z"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "Laravel"
  - "UUID"
---

最近在用 Laravel 5.6 做一个项目，涉及到用户表的自定义字段和 UUID 作为主键，各种 Google 花了我很长时间，所以本篇文章用来记录一下实现思路，以防后人踩坑。

## Schema
用 `php artisan make:auth` 出来的用户表使用的自增的 `id` 作为主键，验证时使用 `email` 字段作为用户的 “登录名”，然而我并不希望使用一个自增的 id，而是使用 UUID 作为用户主键，`user_email` 作为 “登录名”，`user_password` 作为保存的密码。

## UUID
### What’s UUID

对于刚入学时候的萌新开发者来说设计出来的啥数据表都是一个自增 id 主键，根本就不知道 UUID，现在使用上了 UUID 就顺便科普一下啥是 UUID 以及用它有什么好处。

UUID 全称是 Universally Unique Identifier，是一个 128 位的标识符，对外显示是使用 32 位 16 进制的 8-4-4-4-12 位数形式，类似：123e4567-e89b-12d3-a456-426655440000，有一个在线生成 UUID 的网站  [Online UUID Generator](https://www.uuidgenerator.net/) ，读者有兴趣不妨去玩玩～

一般而言主流使用的是 Version 1 和 Version 4 的 UUID，前者使用电脑硬件 MAC 地址，时间戳作为种子，而后者则是完全随机的一个生成过程。

### UUID Collision

既然使用 UUID 作为主键，虽然不像一个自增的 id 那样看上去那么 low，但是还是要考虑碰撞概率问题，毕竟主键撞一下还是爽歪歪的。

UUID 使用的是 122 位的熵，两个 UUID 撞上的概率大约为 10^-37，如果需要找到 p 个碰撞的 UUID 的话，至少需要生成的个数可以由以下公式得出：

![UUID碰撞公式](https://blog-assets.nova.moe/pics/uuid-collision.svg)

所以从理论上来说在使用 UUID 的时候一般不会发生碰撞的事情，但是个人感觉碰撞概率的大小要取决于用的软件，比如 OpenSSL 版本之类的。

### Why UUID

使用 UUID 有什么好处呢？先说说表面上的

1. 隐藏你的真实用户数 举个例子：如果用户看到你的 URL 类似 `submission/23/info` 会怎么想？哦，原来这个平台也就这么点用户数，我才排到 23 呢，而如果使用了 UUID，则可能会是 `submission/840142f2-b248-461c-b16d-2589d03ea028/info` 就完全不会表现出具体的一个排位数量情况，当然，实际实用的时候应该还是 SHA256 然后截取个前 16 位比较看上去舒服一些，类似 `/submission/2e555c22d95bae14/info`，不过这个就不在本文探讨范围内了。
然后是实质上的

2. 对于大型的系统而言，用 UUID 标识一些内容对于统一化而言非常好，类似商品的条形码，如果使用自增的主键的话，长短不一，看上去和 QQ 号一样没品位。

3. 对于分布式系统而言，UUID 可以保证防止出现 id 碰撞，符合分布式系统 CAP 原则的 C(Consistency，强一致性）。

数据结构老师告诉我们，没有任何一种数据结构是全场景最优的，所以这里也要提一下使用 UUID 作为主键的缺点：

1. UUID 往往是使用字符串存储，查询的效率比较低。

2. 不符合 MySQL 官方建议的：If the primary key is long, the secondary indexes use more space, so it is advantageous to have a short primary key.
综上，我们可以很容易想到 UUID 在一些大量的，不需要排序，需要统一化长度的场景中比较适用，如 OJ 的每个用户的提交，商品的唯一标识码等。

## Use UUID as PK
如果不考虑用户表（因为 Laravel 自带的 Auth 很大程度上依赖了自带的数据库表字段）的话，我们要使用 UUID 作为一个表的主键其实非常容易，大致思路如下（使用  [webpatser/laravel-uuid](https://github.com/webpatser/laravel-uuid) ）

### Install UUID
安装 UUID 类

```shell
composer require webpatser/laravel-uuid
```

在 config/app.php 中声明这个类，加入如下行：

```php
'Uuid' => Webpatser\Uuid\Uuid::class,
```

That’s it! 在需要 UUID 字符串的地方 `$uuid = Uuid::generate(4)->string;` 就好了～

### UUID in Laravel

如果你需要在数据表中使用 UUID 作为主键的话，有以下几个事情要做：

1. 数据库 migration 中声明你的主键为 UUID 类型，如下：

```php
$table->primary('new_pk');
$table->uuid(‘new_pk’);
```

2. Model 中需要声明不使用自增的主键和新的主键字段名，如下：

```php
protected $primaryKey = 'new_pk';
public $incrementing = false;
```

3. 为了能在每次执行 create 操作时可以自动创建 UUID 的主键，我们可以重写一下 boot 函数，其中 {$model->getKeyName()} 是用来获取你的主键名称的，在 Model 中：

```php
public static function boot()
{
    parent::boot();
    self::creating(function ($model) {
        $model{$model->getKeyName()} = (string)Uuid::generate(4);
	});
}
```

记得在对应的 Model 顶部加入：

```php
use Webpatser\Uuid\Uuid;
```

这样的话，我们的数据表在新写入数据的时候便会自动生成一个 UUID 作为表的主键了。

### UUID in User Table

如果在用户表中使用了 UUID 的话，事情就会稍微麻烦一点，而对于我的需求 （用户密码字段名称也改了） 的话，就更加麻烦一些了，除了上述操作以外还需要：

1. 定义登录时的 “登录名” 为你自己的字段，比如我用的 `user_email`，在 `app/Http/Controllers/Auth/LoginController.php` 中加入如下：

```php
public function username()
{
    return 'user_email';
}
```

2. 定义登录时的使用的密码字段，比如我使用的是 `user_password`，在 `User.php` 这个 Model 中：

```php
public function getAuthPassword()
{
    return $this->user_password;
}
```

3. 在 `resources/views/auth/login.blade.php` 中将 `email` 相关全部改为新的 “登录名”，对于我的需求就是改成 “user_email”.
以上。

## Reference

1. [Laravel: How can I change the default Auth Password field name](https://stackoverflow.com/a/39382427) 
2. [Setting up UUIDs in Laravel 5+](https://medium.com/@steveazz/setting-up-uuids-in-laravel-5-552412db2088) 
3. [Universally unique identifier - Wikipedia](https://en.wikipedia.org/wiki/Universally_unique_identifier) 
4. [Laravel 使用 UUID 作为用户表主键并使用自定义用户表字段](https://nova.moe/laravel-use-uuid-as-primary-key-with-custom-authentication-fields/)
