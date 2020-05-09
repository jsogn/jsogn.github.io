---
layout: post
title: PAC规则配置
tags:
  - ssr
categories:
  - 科学上网
abbrlink: 57233
---

### 前言

访问github加载过慢，下载项目奇慢无比，开启全局后可以秒下，反应过来github.com默认没有走代理，整理了一些PAC规则的基本配置

<!--more-->

### 基本规则

- 通配符支持，如 `*.example.com/*` 实际书写时可省略 如： `.example.com/` 意即 `.example.com/*`

- 正则表达式支持，以 `\` 开始和结束， 如： `\[\w]+:\/\/example.com\`

- 例外规则 `@@`，如 `@@*.example.com/*` 满足 `@@` 后规则的地址不使用代理

- 匹配地址开始和结尾 `|`，如： `|http://example.com`、`example.com|` 分别表示以 `http://example.com` 开始和以 `example.com` 结束的地址

- || 标记，如： `||example.com` 则 `http://example.com` 、`https://example.com` 、`ftp://example.com` 等地址均满足条件，只用于匹配地址开头

- 注释 `!` 如： `! Comment`

- 分隔符 `^`，表示除了字母、数字或者 `_ - . %` 之外的任何字符如： `http://example.com^` ，`http://example.com/` 和 `http://example.com:8000/` 均满足条件，而 `http://example.com.ar/` 不满足条件

### 自定义规则

```conf
! Put user rules line by line in this file.

! See https://adblockplus.org/en/filter-cheatsheet

@@||localhost

||github.com

```

一行只能有一条代理规则，生效后被配置的域名及其子域名都会经过代理访问
