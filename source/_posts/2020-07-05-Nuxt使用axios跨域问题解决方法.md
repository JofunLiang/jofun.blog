---
title: Nuxt使用axios跨域问题解决方法
date: 2020-07-05
toc: false
comments: false
tags:
    - Vue服务端渲染
categories:
    - Nuxt
---

[Nuxt](https://www.nuxtjs.cn/) 是 Vue 项目服务器端渲染（SSR)解决方案。而在使用时，就会遇到前后端分离情况下的域名或端口不一致导致的跨域问题。本文将介绍如何通过设置代理解决 Nuxt 与 [axios](https://github.com/axios/axios) 集成的跨域问题。

<!--more-->

## 解决跨域

Nuxt 使用 axios 为避免出现前端页面跨域问题，需要安装 @nuxtjs/axios 和 @nuxtjs/proxy 两个模块。

用 yarn 安装：
```
yarn add axios @nuxtjs/axios @nuxtjs/proxy
```
使用 npm 安装：
```
npm install axios @nuxtjs/axios @nuxtjs/proxy
```
> **注意：**不需要手动注册 @nuxtjs/proxy 模块，但是必须要确保它在依赖项中。

安装完成后在 nuxt.config.js 文件里面添加以下配置：
```js
module.exports = {
  /*
   ** Nuxt.js modules
   */
  modules: ["@nuxtjs/axios"],
  /*
   ** axios proxy
   */
  axios: {
    proxy: true
  },
  /*
   ** proxy
   */
  proxy: {
    "/api": "http://localhost:3000"
  },
  /*
   ** Build configuration
   ** See https://nuxtjs.org/api/configuration-build/
   */
  build: {
    vendor: ["axios"]
  }
}
```
到此，代理设置完成，可以测试以下跨域问题是否解决。

## 扩展 axios

创建 nuxt 插件，更改全局配置全局配置自定义 axios，在 plugins/ 目录下新建 axios.js 文件：
```js
// plugins/axios.js
export default function({ $axios, redirect }) {

  $axios.onResponse(res => {
    return res.data
  })

  $axios.onError(error => {
    const code = parseInt(error.response && error.response.status);
    if (code === 400) {
      redirect("/400");
    }
  });
}
```
在 nuxt.config.js 中配置 axios.js 插件：
```js
module.exports = {
  /*
   ** Plugins to load before mounting the App
   ** https://nuxtjs.org/guide/plugins
   */
  plugins: ["@/plugins/axios"],
}
```

## 使用 axios 插件

通过上面的设置后，使用 axios 插件需要注意的是在 asyncData 内和在  asyncData 外的使用是所不同的。

** 在 asyncData 里使用：**
```js
async asyncData({ $axios }) {
  const ip = await $axios.get('http://icanhazip.com')
  return { ip }
}
```

** 在 asyncData 外使用：**
```js
methods: {
  async fetchSomething() {
    const ip = await this.$axios.get('http://icanhazip.com')
    this.ip = ip
  }
}
```

更多关于 Nuxt 与 axios 的集成介绍可以查看官方文档——[Axios模块](https://axios.nuxtjs.org/)。





