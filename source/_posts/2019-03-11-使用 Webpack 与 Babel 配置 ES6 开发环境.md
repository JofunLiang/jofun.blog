---
title: 使用 Webpack 与 Babel 配置 ES6 开发环境
date: 2019-03-11
toc: false
comments: false
tags:
    - Webpack
    - Babel
    - ES6
    - 搭建ES6开发环境
categories:
    - 前端构建工具
---

Webpack 和 Babel 几乎是现在前端开发必备的工具，Webpack 是一个现代 JavaScript 应用程序的静态模块打包器，Babel 是一个 ES6 编译器，由于目前浏览器对 ES6 的兼容性有差异，无法直接在项目中使用 ES6，需要使用 Babel 编译器转换成 ES5 才能在浏览器中运行。使用 Webpack 和 Babel 也开发了几个项目，使用时间少说也有两三年了，本文就讲解一下使用 Webpack 与 Babel 配置 ES6 开发环境。

<!--more-->

## 安装 Webpack

安装：
```
# 本地安装
$ npm install --save-dev webpack webpack-cli

# 全局安装
$ npm install -g webpack webpack-cli
```

在项目根目录下新建一个配置文件—— webpack.config.js 文件：
```javascript
const path = require('path');

module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

在 src 目录下新建 a.js 文件：
```javascript
export const isNull = val => val === null

export const unique = arr => [...new Set(arr)]
```

在 src 目录下新建 index.js 文件：
```javascript
import { isNull, unique } from './a.js'

const arr = [1, 1, 2, 3]

console.log(unique(arr))
console.log(isNull(arr))
```

执行编译打包命令，完成后打开 bundle.js 文件发现 isNull 和 unique 两个函数没有被编译，和 webpack 官方说法一致：webpack 默认支持 ES6 模块语法，要编译 ES6 代码依然需要 babel 编译器。

## 安装配置 Babel 编译器

使用 Babel 必须先安装 @babel/core 和 @babel/preset-env 两个模块，其中 @babel/core 是 Babel 的核心存在，Babel 的核心 api 都在这个模块里面，比如：transform。而 @babel/preset-env 是一个智能预设，允许您使用最新的 JavaScript，而无需微观管理您的目标环境需要哪些语法转换（以及可选的浏览器polyfill）。因为这里使用的打包工具是 Webpack，所以还需要安装 babel-loader 插件。

安装：
```
$ npm install --save-dev @babel/core @babel/preset-env babel-loader
```

新建 .babelrc 文件：
```json
{
  "presets": [
    "@babel/preset-env"
  ]
}
```

修改 webpack 配置文件（webpack.config.js）：
```javascript
const path = require('path');

module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
          loader: 'babel-loader',
          exclude: /node_modules/
      }
    ]
  }
}
```

由于 babel 默认只转换 ES6 新语法，不转换新的 API，如：Set、Map、Promise等，所以需要安装 @babel/polyfill 转换新 API。安装 @babel/plugin-transform-runtime 优化代码，@babel/plugin-transform-runtime 是一个可以重复使用 Babel 注入的帮助程序代码来节省代码的插件。

安装 @babel/polyfill、@babel/plugin-transform-runtime 两个插件：
```
$ npm install --save-dev @babel/polyfill @babel/plugin-transform-runtime
```

修改 .babelrc 配置文件：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "useBuiltIns": "usage", // 在每个文件中使用polyfill时，为polyfill添加特定导入。利用捆绑器只加载一次相同的polyfill。
      "modules": false // 启用将ES6模块语法转换为其他模块类型，设置为false不会转换模块。
    }]
  ],
  "plugins": [
    ["@babel/plugin-transform-runtime", {
      "helpers": false
    }]
  ]
}
```

最后，配置兼容的浏览器环境。在 .babelrc 配置文件中设置 targets 属性：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "useBuiltIns": "usage",
      "modules": false,
      "targets": {
        "browsers": "last 2 versions, not ie <= 9"
      }
    }]
  ],
  "plugins": [
    ["@babel/plugin-transform-runtime", {
      "helpers": false
    }]
  ]
}
```

执行命令编译代码，完成后检查 bundle.js 文件，是否成功转换新 API 。如果发现以下代码即说明转换成功：
```javascript
// 23.2 Set Objects
module.exports = __webpack_require__(80)(SET, function (get) {
  return function Set() { return get(this, arguments.length > 0 ? arguments[0] : undefined); };
}, {
  // 23.2.3.1 Set.prototype.add(value)
  add: function add(value) {
    return strong.def(validate(this, SET), value = value === 0 ? 0 : value, value);
  }
}, strong);
```

其他关于 js 压缩和 Webpack 启用 tree shaking 功能的设置本文不在赘述。

## 配置文件详情概览

package.json 文件：
```javascript
{
  "name": "demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.3.4",
    "@babel/plugin-transform-runtime": "^7.3.4",
    "@babel/polyfill": "^7.2.5",
    "@babel/preset-env": "^7.3.4",
    "babel-loader": "^8.0.5",
    "webpack": "^4.29.6",
    "webpack-cli": "^3.2.3"
  }
}
```

webpack.config.js 文件：
```javascript
const path = require('path');

module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
          loader: 'babel-loader',
          exclude: /node_modules/
      }
    ]
  }
}
```

.babelrc 文件：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "useBuiltIns": "usage",
      "modules": false,
      "targets": {
        "browsers": "last 2 versions, not ie <= 9"
      }
    }]
  ],
  "plugins": [
    ["@babel/plugin-transform-runtime", {
      "helpers": false
    }]
  ]
}
```
