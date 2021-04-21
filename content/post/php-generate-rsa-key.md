---
title: "使用php生成RSA公钥私钥及进行加密/解密/签名/验证"
slug: "php-generate-rsa-key"
description: "这篇文章主要介绍使用PHP开发接口，数据实现RSA加密解密后使用，实例分析了PHP自定义RSA类实现加密与解密的技巧，非常具有实用价值，需要的朋友可以参考下。"
date: "2018-10-19T23:41:42+08:00"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "php"
  - "rsa"
  - "openssl"
---
这篇文章主要介绍使用PHP开发接口，数据实现RSA加密解密后使用，实例分析了PHP自定义RSA类实现加密与解密的技巧，非常具有实用价值，需要的朋友可以参考下。

简单介绍RSA：

RSA加密算法是最常用的非对称加密算法，CFCA在证书服务中离不了它。但是有不少新手对它不太了解。下面仅作简要介绍。RSA是第一个比较完善的公开密钥算法，它既能用于加密，也能用于数字签名。RSA以它的三个发明者Ron Rivest, Adi Shamir, Leonard Adleman的名字首字母命名，这个算法经受住了多年深入的密码分析，虽然密码分析者既不能证明也不能否定RSA的安全性，但这恰恰说明该算法有一定的可信性，目前它已经成为最流行的公开密钥算法。RSA的安全基于大数分解的难度。其公钥和私钥是一对大素数（100到200位十进制数或更大）的函数。从一个公钥和密文恢复出明文的难度，等价于分解两个大素数之积（这是公认的数学难题）。

下面为具体类：Rsa.class.php

```php
<?php

/**
 * RSA算法类
 * 签名及密文编码：base64字符串/十六进制字符串/二进制字符串流
 * 填充方式: PKCS1Padding（加解密）/NOPadding（解密）
 *
 * Notice:Only accepts a single block. Block size is equal to the RSA key size!
 * 如密钥长度为1024 bit，则加密时数据需小于128字节，加上PKCS1Padding本身的11字节信息，所以明文需小于117字节
 */
class RSA
{
    private $pubKey = null;
    private $priKey = null;

    /**
     * 构造函数
     */
    public function __construct()
    {
        // 需要开启openssl扩展
        if (!extension_loaded("openssl")) {
            $this->_error("Please open the openssl extension first.");
        }
    }

    /**
     * 读取公钥和私钥
     * @param string $public_key_file 公钥文件（验签和加密时传入）
     * @param string $private_key_file 私钥文件（签名和解密时传入）
     */
    public function init($public_key_file = '', $private_key_file = '')
    {
        if ($public_key_file) {
            $this->_getPublicKey($public_key_file);
        }

        if ($private_key_file) {
            $this->_getPrivateKey($private_key_file);
        }
    }

    /**
     * 自定义错误处理
     */
    private function _error($msg)
    {
        die('RSA Error:' . $msg); //TODO
    }

    /**
     * 检测填充类型
     * 加密只支持PKCS1_PADDING
     * 解密支持PKCS1_PADDING和NO_PADDING
     *
     * @param int 填充模式
     * @param string 加密en/解密de
     * @return bool
     */
    private function _checkPadding($padding, $type)
    {
        if ($type == 'en') {
            switch ($padding) {
                case OPENSSL_PKCS1_PADDING:
                    $ret = true;
                    break;
                default:
                    $ret = false;
            }
        } else {
            switch ($padding) {
                case OPENSSL_PKCS1_PADDING:
                case OPENSSL_NO_PADDING:
                    $ret = true;
                    break;
                default:
                    $ret = false;
            }
        }
        return $ret;
    }

    private function _encode($data, $code)
    {
        switch (strtolower($code)) {
            case 'base64':
                $data = base64_encode('' . $data);
                break;
            case 'hex':
                $data = bin2hex($data);
                break;
            case 'bin':
            default:
        }
        return $data;
    }

    private function _decode($data, $code)
    {
        switch (strtolower($code)) {
            case 'base64':
                $data = base64_decode($data);
                break;
            case 'hex':
                $data = $this->_hex2bin($data);
                break;
            case 'bin':
            default:
        }
        return $data;
    }

    private function _getPublicKey($file)
    {
        $key_content = $this->_readFile($file);
        if ($key_content) {
            $this->pubKey = openssl_get_publickey($key_content);
        }
    }

    private function _getPrivateKey($file)
    {
        $key_content = $this->_readFile($file);
        if ($key_content) {
            $this->priKey = openssl_get_privatekey($key_content);
        }
    }

    private function _readFile($file)
    {
        $ret = false;
        if (!file_exists($file)) {
            $this->_error("The file {$file} is not exists");
        } else {
            $ret = file_get_contents($file);
        }
        return $ret;
    }

    private function _hex2bin($hex = false)
    {
        $ret = $hex !== false && preg_match('/^[0-9a-fA-F]+$/i', $hex) ? pack("H*", $hex) : false;
        return $ret;
    }

    /**
     * 生成Rsa公钥和私钥
     * @param int $private_key_bits 建议：[512, 1024, 2048, 4096]
     * @return array
     */
    public function generate(int $private_key_bits = 1024)
    {
        $rsa = [
            "private_key" => "",
            "public_key" => ""
        ];

        $config = [
            "digest_alg" => "sha512",
            "private_key_bits" => $private_key_bits, #此处必须为int类型
            "private_key_type" => OPENSSL_KEYTYPE_RSA,
        ];

        //创建公钥和私钥
        $res = openssl_pkey_new($config);

        //提取私钥
        openssl_pkey_export($res, $rsa['private_key']);

        //生成公钥
        $rsa['public_key'] = openssl_pkey_get_details($res)["key"];
        /*Array
        (
            [bits] => 512
            [key] =>
            [rsa] =>
            [type] => 0
        )*/
        return $rsa;
    }

    /**
     * 生成签名
     *
     * @param string 签名材料
     * @param string 签名编码（base64/hex/bin）
     * @return bool|string 签名值
     */
    public function sign($data, $code = 'base64')
    {
        $ret = false;
        if (openssl_sign($data, $ret, $this->priKey)) {
            $ret = $this->_encode($ret, $code);
        }
        return $ret;
    }

    /**
     * 验证签名
     *
     * @param string 签名材料
     * @param string 签名值
     * @param string 签名编码（base64/hex/bin）
     * @return bool
     */
    public function verify($data, $sign, $code = 'base64')
    {
        $ret = false;
        $sign = $this->_decode($sign, $code);
        if ($sign !== false) {
            switch (openssl_verify($data, $sign, $this->pubKey)) {
                case 1:
                    $ret = true;
                    break;
                case 0:
                case -1:
                default:
                    $ret = false;
            }
        }
        return $ret;
    }

    /**
     * 加密
     *
     * @param string 明文
     * @param string 密文编码（base64/hex/bin）
     * @param int 填充方式（貌似php有bug，所以目前仅支持OPENSSL_PKCS1_PADDING）
     * @return string 密文
     */
    public function encrypt($data, $code = 'base64', $padding = OPENSSL_PKCS1_PADDING)
    {
        $ret = false;
        if (!$this->_checkPadding($padding, 'en')) $this->_error('padding error');
        if (openssl_public_encrypt($data, $result, $this->pubKey, $padding)) {
            $ret = $this->_encode($result, $code);
        }
        return $ret;
    }

    /**
     * 解密
     *
     * @param string 密文
     * @param string 密文编码（base64/hex/bin）
     * @param int 填充方式（OPENSSL_PKCS1_PADDING / OPENSSL_NO_PADDING）
     * @param bool 是否翻转明文（When passing Microsoft CryptoAPI-generated RSA cyphertext, revert the bytes in the block）
     * @return string 明文
     */
    public function decrypt($data, $code = 'base64', $padding = OPENSSL_PKCS1_PADDING, $rev = false)
    {
        $ret = false;
        $data = $this->_decode($data, $code);
        if (!$this->_checkPadding($padding, 'de')) $this->_error('padding error');
        if ($data !== false) {
            if (openssl_private_decrypt($data, $result, $this->priKey, $padding)) {
                $ret = $rev ? rtrim(strrev($result), "\0") : '' . $result;
            }
        }
        return $ret;
    }
}
```

RSA类的使用：use.php

```php
<?php

if (!class_exists("RSA")) {
    require_once "Rsa.class.php";
}

$private_key_file = __DIR__ . "/cert/private_key.pem";
$public_key_file = __DIR__ . "/cert/public_key.pem";

$rsa = new RSA();

// 没有就生成一对
if (!file_exists($private_key_file) || !file_exists($public_key_file)) {
    $key = $rsa->generate();

    file_put_contents($private_key_file, $key['private_key'], LOCK_EX);
    file_put_contents($public_key_file, $key['public_key'], LOCK_EX);
} else {
    $key = [
        "private_key" => file_get_contents($private_key_file),
        "public_key" => file_get_contents($public_key_file)
    ];
}

//显示数据
echo "private_key:\n" . $key['private_key'] . "\n\r";
echo "public_key:\n" . $key['public_key'] . "\n\r";

//要加密的数据
$data = "Web site:http://blog.kilvn.com";
echo '加密的数据：' . $data, "\n-------------------------------\n";

$rsa->init($public_key_file, $private_key_file);


//加密
$encrypt = $rsa->encrypt($data);
echo "公钥加密后的数据: " . $encrypt . "\n";

//解密
$decrypt = $rsa->decrypt($encrypt);
echo "私钥解密后的数据: " . $decrypt, "\n-------------------------------\n";

//签名
$sign = $rsa->sign($data);
echo "签名的数据: " . $sign . "\n";

//验证
$verify = $rsa->verify($data, $sign);
echo "验证的数据: " . $verify . "\n", "\n-------------------------------\n";
```

运行结果如下：

```
/usr/bin/php /Volumes/Mac软件/WEB/www/use.php
private_key:
-----BEGIN PRIVATE KEY-----
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
mpPQ1kaLJ7QoBD/vT74UhxpqUgCma/Q=
-----END PRIVATE KEY-----

public_key:
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAwGji+aUGELbI6sa7ifb0
W9QYCp9aMSy0zg5TXV/fDmg/9/hAqjRpTeLThLSxeTNPmzf2VZ9jvrZ7IjkYldfA
2VOe4x91DosU1Fniwf6TYuNemM3H/DK9dGuFXzI1jsiiAobyd5Jb5Ptt0HWLgKNb
jXquh8UddadHgaeKwlbtYYIypb9hhf4ocywQc2dbBBS6e7RpI/XQr+TDkhZPvUYa
4lAaDnPdbc9V75qE/WU9Q2ACiw6jtrcpL1nH5Hif4OHrAHrKLOUyUhUXdkKDFjxN
aZfXxDEUqxLkUw7PXlmww3Rcr3S5MQVP9UXqgHlvOQOYiddXYY81XKA5pXKcWIrA
xjbeUPmVW7UmCFOlYcV/j+rVNDMokQqUragMi+pqpYkzXum7UXB5zjsB8kMVx8l4
/0M5vNk4XGzFCbj4pdsLwxRFW7AVqHqRiGYEEEkl/UpHIl+T+aEZXQI/uAa0q9+R
BoqXVqMqmoYT6R5wPUOliDYchAQUxJkGVow60Y7kVrtIlqPvOG4ugLLpKRTiT+Tz
4mtmMEX+N+UmrAKABJUiGj7JD/lLPWAIKvOx+WlB47k2bTVZp5yTweezU0nOnaU5
gdoWqemWo5EC7JXO73ItLcKGQIY3qUu9tUTXUKTDO/XiDvPP04ggE9bODLfvoSBS
Ph5ANrtx0wxRMeu+VNoeqx8CAwEAAQ==
-----END PUBLIC KEY-----

加密的数据：Web site:http://blog.kilvn.com
-------------------------------
公钥加密后的数据: WJ3JtHA9h+fJyPgl4WNtmplqexi6AvBCP0yC4zFo8AsgHyIu2u5Jt/olDw8Hmjmy4++XjdJwlKC9pNwNTiuj/TvDoWAulDJSgmrTZ/Jrp78K8Puik8n3Zudtmqj9rMMKAwgaX4t8+QRA+tBs5HiRUBpDsKPL8rejqIYZwMwxf67ldbwWFUtuFb8OWvG6EeQjPoKV2mOWi7/KnNFTLc6970sWTgcVe53KeqWrNyHvHLhQda3uom8lf1eOwOrOmhtI1sQdihH36LATCzObcDDsRTJr2oW3Y2d1Sx1npVJ92AvOmOVxFcena8ppkZ+cCRWAxTbiZejdLv9dGCn227yEm5/A5b1MXSHxZcXQ56Vy56Lo735+G6V5zkupVbQJrZBAJg0iDJPi6DFzffLMlTmRlszmiGxg67lxbKQQMhpYkVgZ9PfgYmXzUPVZSXRCz7zGkrP7qqAmbA2tGxRadh1FgekDmb8lSYEbLP6w1gKv5eJTCWYAeUTYuSHsLvCzF2+rJfRz9XrYYVaYAz5ECPCSCOSldENu97Ol612FWJuLZZNFiTLfpkqTYIvgSVGnqbuIMxU+HIifC+/cHabwSpvJ7q2lGmtdcv5mmHfi2V2XpB7SRfe67sbBIeIhF0XOqFoOf1VuhsC+ZP0INV1DIQVvTmivl5+IrORhrxc1c090Eso=
私钥解密后的数据: Web site:http://blog.kilvn.com
-------------------------------
签名的数据: enU5lIt8t7R5nsWw0OK7O8n4yttwo4cR44MDCqr8l4CmobLKgsFJyNskY33EtusisWST8UQconXtRxscEWrFXZTO74xsLKVLP1ZffJZyeRB0hiygNk/h3rZgtZXzGvMDBuOHBn40f4x+DQ83eZuvNPNNjA6nbuoBvyKT7WiP3bQ9HeJQBj7hfQhHbGYnOLwXTvXUbJDhDfpX6Ux4pAz9rjV14s9eKyfmncH0gIbHQDpMd+B06cdKi8i3pX2tGzPleEw9XRIfu1fkDxZ4J9ZruTUDUU0V5jhi+YlBFDo7Hsj3wsAVPTwOIgoRVwggaFCNeXLqx2u0u4BHj0FxJ6Qk2CC8izhcSoVQCfhrS3g9rUP5jCX4JP62kPJ1tIrxx2T+xs2tbdppe7q4Wx6EuIlZvWpnc7oHmPRP2Z3wEfsn49n8wI2f0zSYq3YmyrO0yEFT5bpvLggf6eyIsnnOO/uI/QfksAzSoKuJ3Wf6v0jYZrij/Y6d0jXIkn1MQpFjk5ZA4Dajt2frAzPXViqgdO8YuMFCFD5U2V6GnSYQ8xLR1V88zm3bH/baqqQFOMV0wMaZh9UtgBT5mCRXnUtYHdSjAhfj9xygOtw61p66EKO4Ykf7yxpivOI1QNn1hzA52MPktOTJwitUQXvSUDB5mgkjCPbS05LitAGU71REQ1urk5w=
验证的数据: 1

-------------------------------

Process finished with exit code 0
```

相关参考：

* [PHP开发接口使用RSA进行加密解密方法](https://www.cnblogs.com/jiangxiaobo/p/7843671.html)

* [使用php生成RSA公钥私钥及进行加密解密示例](http://www.04007.cn/article/259.html)

* [非对称加解密，私钥和公钥到底是谁来加密，谁来解密](https://blog.csdn.net/qq_23167527/article/details/80614454)

