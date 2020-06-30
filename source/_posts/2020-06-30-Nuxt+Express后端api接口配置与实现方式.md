---
title: Nuxt+Express后端api接口配置与实现方式
date: 2020-06-30
toc: false
comments: false
tags:
    - Vue服务端渲染
categories:
    - Nuxt
---

[Nuxt.js](https://www.nuxtjs.cn/) 是一个基于 Vue.js 的轻量级应用框架,可用来创建服务端渲染 (SSR) 应用。本文带你了解在 Nuxt.js 中使用 Express 如何编写实现后端的 api 接口。

<!--more-->

## 创建接口文件

在项目根目录中新建 server 文件夹并在该文件夹下创建一个 index.js 文件。
```
server
└── index.js
```

然后，在 server/index.js 中使用 Express 创建服务器路由中间件，以下创建一个返回字符串 ‘Hello World!’ 的简单接口示例。
```js
const app = require('express')();

app.get('/hello', (req, res) => {
  res.send('Hello World!')
})

module.exports = {
  path: 'api',
  handler: app
}
```

接下来，修改 nuxt.config.js 文件，在 serverMiddleware 配置项中添加 api 中间件。
```js
module.exports = {
  serverMiddleware: [
    // API middleware
    '~/server/index.js'
  ],
}
```

现在，重启服务：
```
npm run dev
```
启动后，在浏览器地址栏中输入 http://localhost:3000/api/hello 查看是否成功返回 ‘Hello World!’。

对于如何注册第三方路由的详细用法请查看 nuxt.config.js 配置文档[serverMiddleware](https://nuxtjs.org/api/configuration-servermiddleware)属性的介绍。


## 在页面中请求 api 数据

Nuxt.js添加了一种 asyncData 方法，可让您在初始化组件之前处理异步操作。asyncData 每次在加载页面组件之前都会调用。此方法将上下文作为第一个参数，您可以使用它来获取一些数据，Nuxt.js 会将其与组件数据合并。

修改 api/hello 接口，使之返回 JSON 数据。
```js
app.get('/hello', (req, res) => {
  res.json({
    title: 'Hello World!'
  })
})
```

在 pages/index.vue 中请求上面修改完成的 api/hello 接口。
```js
export default {
  asyncData () {
    return fetch('http://localhost:3000/api/hello', { method: 'GET' })
      .then(res => res.json())
      .then(res => {
        // asyncData 方法获取的数据会与组件数据合并，所以这里返回对象
        return {
          title: res.title
        }
      })
  }
}
```

接下来只需在 template 中绑定 title 即可显示请求返回的数据。
```html
<template>
  <h1>{{ title }}<h1>
</template>
```

关于异步获取数据请移步文档[asyncData](https://nuxtjs.org/guide/async-data)的介绍。