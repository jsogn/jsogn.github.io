---
layout: post
title: 轻松生成jwt的插件
tags:
  - php
categories:
  - php
abbrlink: 60442
date: 2019-05-28 16:50:45
---


### 前言

json web token，无需多说

<!--more-->

### 安装

```shell
composer require lcobucci/jwt
```

### 参数说明

- iss 【issuer】发布者的url地址
- aud 【audience】接受者的url地址
- sub 【subject】该JWT所面向的用户，用于处理特定应用，不是常用的字段
- exp 【expiration】 该jwt销毁的时间；unix时间戳
- nbf 【not before】 该jwt的使用时间不能早于该时间；unix时间戳
- iat 【issued at】 该jwt的发布时间；unix 时间戳
- jti 【JWT ID】该jwt的唯一ID编号


### 生成token

```php
<?php

require './vendor/autoload.php';

use Lcobucci\JWT\Builder;
use Lcobucci\JWT\Signer\Hmac\Sha256;
use Lcobucci\JWT\Signer\Key;

//发布端url
$iss = 'http://admin.jwt.com';
//请求端URL
$aud = 'http://api.jwt.com/user/login';
//唯一的jwt id
$jwt_id = 'jwt123';
//私钥，用于token验证
$signer_key = 'jwt-test';

$signer = new Sha256();

$token = (new Builder())->issuedBy($iss)
                        ->permittedFor($aud)
                        ->identifiedBy($jwt_id, true)
                        ->issuedAt(time())
                        ->canOnlyBeUsedAfter(time() + 60)
                        ->expiresAt(time() + 3600)
                        ->set('user', ['name' => 'police', 'mobile' => '110'])
                        ->withClaim('uid', 1)
                        ->getToken($signer, new Key($signer_key));
echo $token;

```

### 验证token

```php

<?php
require './vendor/autoload.php';

use Lcobucci\JWT\ValidationData;
use Lcobucci\JWT\Parser;
use Lcobucci\JWT\Signer\Hmac\Sha256;

$token  = $_GET['token'] ?? '';
$token  = (new Parser())->parse((string) $token);
$signer = new Sha256();

$aud    = $token->getClaim('aud');
$iss    = $token->getClaim('iss');
$jwt_id = $token->getHeader('jti');
$user   = $token->getClaim('user');


$signer_key = 'jwt-test';  //私钥

$data = new ValidationData();

$data->setIssuer($iss);
$data->setAudience($aud);
$data->setId($jwt_id);

//验证私钥
var_dump($token->verify($signer, $signer_key));

//失败，因为token在60秒后方可验证
var_dump($token->validate($data));

//修改验证时间
$data->setCurrentTime(time() + 60);

// true
var_dump($token->validate($data));

$data->setCurrentTime(time() + 4000);

//false,token过期
var_dump($token->validate($data));

```