---
layout: post
title: vue配置跨域代理
tags:
  - vue
categories:
  - vue
abbrlink: 42883
---

### 前言

如果前端应用和后端 API 服务器没有运行在同一个主机上，需要在开发环境下将 API 请求代理到 API 服务器。这个问题可以通过 vue.config.js 中的 devServer.proxy 选项来配置。

<!--more-->

### vue.config.js

`vue.config.js` 是一个可选的配置文件，如果项目的 (和 package.json 同级的) 根目录中存在这个文件，那么它会被 @vue/cli-service 自动加载。你也可以使用 package.json 中的 vue 字段，但是注意这种写法需要你严格遵照 JSON 的格式来写。

这个文件应该导出一个包含了选项的对象

```js
// vue.config.js
module.exports = {
  // 选项...
}
```

### devServer.proxy

devServer.proxy 可以是一个指向开发环境 API 服务器的字符串：

```js
module.exports = {
  devServer: {
    proxy: 'http://localhost:4000'
  }
}
```

这会告诉开发服务器将任何未知请求 (没有匹配到静态文件的请求) 代理到http://localhost:4000。

```js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: '<url>',
        ws: true,
        changeOrigin: true
        pathRewrite: {
            '^/api': ''
        }
      },
      '/foo': {
        target: '<other_url>'
      }
    }
  }
}
```

更多的代理控制行为，也可以使用一个 path: options 成对的对象。完整的选项可以查阅 [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware#proxycontext-config) 。

### 总结

以上均出自官方文档，发现很多人不知道看vue配置文档，百分之八十的问题都可以通过文档解决
