---
title: Vue CLI 3配置svg-sprite-loader与svgo-loader
date: 2019-11-16
toc: false
comments: false
tags:
    - Vue CLI 3
    - svg-sprite-loader
    - svgo-loader
    - svgo
    - svg symbols
    - web图标
categories:
    - SVG
---

随着高清屏幕的普及，相比使用png等位图而言，使用SVG等矢量图形是一种全新的设计方式。更重要的是相比位图而言，SVG有着无可比拟的优势。其中，使用SVG中的[Symbol元素](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/symbol)制作Icon图标变得越来越流行，这种技术被称为SVG Sprite。这里所说的Sprite技术，类似于CSS中的Sprite技术。图标图形整合在一起，实际呈现的时候准确显示特定图标。

而Vue CLI已经更新到了3.0版本，那么[Vue CLI 3.0](https://cli.vuejs.org/zh/guide/) 如何修改Webpack设置来配置[svg-sprite-loader](https://github.com/JetBrains/svg-sprite-loader)与[svgo-loader](https://github.com/rpominov/svgo-loader)实现SVG Sprite？

<!--more-->

## 配置svg-sprite-loader

安装svg-sprite-loader：
```
npm install --save-dev svg-sprite-loader
```

Vue CLI 3.0 与 2.x版本不同，3.0版本内部的 webpack 配置是通过 webpack-chain 维护的，因此，你需要熟悉 [webpack-chain 的 API](https://github.com/neutrinojs/webpack-chain)。

Vue CLI对svg有一个默认的rule设置，这个设置通过file-loader加载svg文件。因此，需要在vue.config.js中先清除默认的svg配置：
```js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule("svg")
      .uses
      .clear()
      .end()
  }
}
```
接着，在vue.config.js中配置svg-sprite-loader，将处理路径设置为你的svg图标路径（如：“./src/icons”），方法如下：
```js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule("svg")
      .uses
      .clear()
      .end()
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]",
        include: ["./src/icons"]
      })
      .end()
  }
}
```

## 配置svgo-loader

安装svgo和svgo-loader：
```
npm install svgo svgo-loader --save-dev
```
在vue.config.js中加上svgo-loader的配置：
```js
module.exports = {
  chainWebpack: config => {
    config.module
      .rule("svg")
      .uses
      .clear()
      .end()
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]",
        include: ["./src/icons"]
      })
      .end()
      .before("svg-sprite-loader")
      .use("svgo-loader")
      .loader("svgo-loader")
      .options({
        plugins: [
          {removeAttrs: {attrs: "path:fill"}}
        ]
      })
      .end()
  }
}
```
svgo-loader加载器options的plugins配置项参考[svgo配置项列表](https://github.com/svg/svgo#what-it-can-do)。

## 优化配置

上面直接清除默认的svg配置可能会引起异常，应该排除“./src/icons”这个svg图标文件夹即可，方法如下：
```js
const { resolve } = require('path')

module.exports = {
  chainWebpack: config => {
    // 清除svg默认配置对./src/icons文件夹的处理
    config.module
      .rule("svg")
      .exclude
      .add(resolve("./src/icons"))
      .end()
  }
}
```
接下来，对“./src/icons”添加一个新的rule，配置如下：
```js
const { resolve } = require('path')

module.exports = {
  chainWebpack: config => {
    // 添加新的rule处理./src/icons文件夹的svg文件
    config.module
      .rule("svg-sprite")
      .test(/\.svg$/)
      .include
      .add(resolve("./src/icons"))
      .end()
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]"
      })
      .end()
      .before("svg-sprite-loader")
      .use("svgo-loader")
      .loader("svgo-loader")
      .options({
        plugins: [
          {removeAttrs: {attrs: "path:fill"}}
        ]
      })
      .end()
  }
}
```
完整的配置如下：
```js
const { resolve } = require('path')

module.exports = {
  chainWebpack: config => {
    // 清除svg默认配置对./src/icons文件夹的处理
    config.module
      .rule("svg")
      .exclude
      .add(resolve("./src/icons"))
      .end()
    
    // 添加新的rule处理./src/icons文件夹的svg文件
    config.module
      .rule("svg-sprite")
      .test(/\.svg$/)
      .include
      .add(resolve("./src/icons"))
      .end()
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]"
      })
      .end()
      .before("svg-sprite-loader")
      .use("svgo-loader")
      .loader("svgo-loader")
      .options({
        plugins: [
          {removeAttrs: {attrs: "path:fill"}}
        ]
      })
      .end()
  }
}
```
