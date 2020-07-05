---
title: Nuxt配置Element-UI按需引入方法
date: 2020-07-04
toc: false
comments: false
tags:
    - Vue服务端渲染
categories:
    - Nuxt
---

[Nuxt.js](https://www.nuxtjs.cn/) 使用 create-nuxt-app 创建项目时，选择使用 Element-UI 为默认组件库，发现 Nuxt 没有开启 Element-UI 的按需引入配置，需要自行配置。

<!--more-->

## 安装依赖

在 create-nuxt-app 中没有选择 Element-UI 的先安装。
```
npm install element-ui --save
```
或者
```
yarn add element-ui
```

Element-UI 开启按需引入，必须安装 babel-plugin-component 插件。
```
npm install babel-plugin-component --save-dev
```
或者
```
yarn add babel-plugin-component
```

安装完成后，在文件根目录创建(或已经存在) plugins/ 目录下创建相应的插件文件，创建名为：element-ui.js 的文件。
```js
// element-ui.js
import Vue from 'vue'
import {
  Container,
  Header,
  Aside,
  Main,
  Menu,
  MenuItem,
  Button,
  Form,
  FormItem,
  Input
} from 'element-ui'
import locale from 'element-ui/lib/locale/lang/en'

const components = [
  Container,
  Header,
  Aside,
  Main,
  Menu,
  MenuItem,
  Button,
  Form,
  FormItem,
  Input
];

const Element = {
  install (Vue) {
    components.forEach(component => {
      Vue.component(component.name, component)
    })
  }
}

Vue.use(Element, { locale })
```

## 配置 plugins 选项

在 nuxt.config.js 文件中，配置 plugins 选项。
```js
module.exports = {
  /*
   ** Plugins to load before mounting the App
   ** https://nuxtjs.org/guide/plugins
   */
  plugins: ["@/plugins/element-ui"],
}
```

Nuxt 默认为开启 SSR，采用服务端渲染，也可以手动配置关闭 SSR，配置为：
```js
module.exports = {
  /*
   ** Plugins to load before mounting the App
   ** https://nuxtjs.org/guide/plugins
   */
  plugins: [
    {
      src: "@/plugins/element-ui",
      ssr: false  // 关闭ssr
    }
  ],
}
```

如果在 create-nuxt-app 中默认选了 Element-UI 的，还需要将 Element-UI 的全局样式去掉，即在 nuxt.config.js 中：
```js
module.exports = {
  /*
   ** Global CSS
   */
  css: ['element-ui/lib/theme-chalk/index.css'],
}
```
删除 'element-ui/lib/theme-chalk/index.css' 作为全局样式的打包配置，改为
```js
module.exports = {
  /*
   ** Global CSS
   */
  css: [],
}
```


## 配置 babel 选项

在 nuxt.config.js 文件中，在选项 build 中配置 babel 选项：
```js
module.exports = {
  /*
   ** Build configuration
   ** See https://nuxtjs.org/api/configuration-build/
   */
  build: {
    babel: {
      "plugins": [
        [
          "component",
          {
            "libraryName": "element-ui",
            "styleLibraryName": "theme-chalk"
          }
        ]
      ]
    }
  }
}
```

到此，Element-UI 按需引入配置完成。


