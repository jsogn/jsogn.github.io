---
layout: post
title: Nginx访问控制
tags:
  - nginx
categories:
  - nginx
abbrlink: 16692
---
### 前言

有时，网站会被恶意入侵，可用Nginx做一些访问控制，加强一些网站安全性

<!--more-->

### 扩展名限制

禁止访问指定目录下的程序

```conf
location ~ ^/images/.*\.(php|py)$
{
    deny all;
}

```

禁止访问指定文件名程序

```conf
location ~ ^/data/(attachment|avatar).*\.(php|py)$
{
    deny all;
}
```


### 文件或目录限制

```conf
location ~ ^/(\.user.ini|\.htaccess|\.git|\.svn|\.project|LICENSE|README.md) {
    return 404;
}
```

排除某个目录不受限制

```conf
location ~ \.well-known{
    allow all;
}
```

禁止访问单个目录

```conf
location ~ ^/static {
    deny all;
}
```

禁止访问多个目录，并返回指定状态码

```conf
location ~ ^/(static|js) {
    return 404;
}
```

### 限制IP访问

禁止目录访问，但允许某IP访问

```conf
location ~ ^/mysql_loging/ {
    allow 192.168.0.4;
    deny all;
}
```

限制IP或IP段访问

```conf
location / {
    deny 192.168.0.4;
    allow 192.168.1.0/16;
    allow 10.0.0.0/24;
    deny all;
}
```

nginx做反向代理的时候也可以限制客户端IP

```conf
if ( $remoteaddr = 10.0.0.7 ) {
    return 403;
}

if ( $remoteaddr = 218.247.17.130 ) {
    set $allow_access_root 'ture';
}
```

禁止某ip段访问并向浏览器输出一段文字（若有乱码,请在server中添加：charset utf-8;）

```conf
if ($remote_addr ~* ^211\.149\.(.*?)\.(.*?)$){
return 404 "黑名单用户，拒绝访问";
}
```

### 禁止非法域名解析访问

让使用IP访问网站的用户，或恶意接卸域名的用户，收到501错误

```conf
server {
    listen 80 default_server;
    server_name _;
    return 501;
}
```

通过301跳转主页

```conf
server {
    listen 80 default_server;
    server_name _;
    rewrite ^(.*) http://blog.dns.com/$1 permanent;
}
```

发现某域名恶意解析到公司的服务器IP，在server标签里添加以下代码即可，若有多个server要多处添加

```conf
if ($host !~ ^www/.tag/.com$) {
    rewrite ^(.*) http://blog.mydns.vip$1 permanent;
}
```