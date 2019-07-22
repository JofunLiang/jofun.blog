---
title: 使用SVG symbols建立图标系统完整指南
date: 2019-07-21
toc: false
comments: false
tags:
    - web图标
    - symbols
    - webpack
    - gulp
categories:
    - SVG
---

从最开始的使用img图片，到后来的使用css sprite来减少服务器请求，再到流行的图形字体化图标Iconfont。现在，一种全新的图标使用方式开始流行了起来——SVG symbols图标。

<!--more-->

## 工作原理

SVG symbols的工作原理：symbol元素用来定义一个图形模板对象，它可以用一个use元素实例化。

symbol元素对图形的作用是在同一文档中多次使用，symbol元素本身是不呈现的。只有symbol元素的实例（亦即，一个引用了symbol的use元素）才能呈现：
```html
<svg>
  <symbol viewBox="0 0 24 24" id="heart">
    <path fill="#E86C60" d="M17,0c-1.9,0-3.7,0.8-5,2.1C10.7,0.8,8.9,0,7,0C3.1,0,0,3.1,0,7c0,6.4,10.9,15.4,11.4,15.8 c0.2,0.2,0.4,0.2,0.6,0.2s0.4-0.1,0.6-0.2C13.1,22.4,24,13.4,24,7C24,3.1,20.9,0,17,0z">
    </path>
  </symbol>
  <symbol viewBox="0 0 32 32" id="arrow">
    <path fill="#0f0f0f" d="M16,0C7.2,0,0,7.2,0,16s7.2,16,16,16s16-7.2,16-16S24.8,0,16,0z M22.8,13.6l-6,8C16.6,21.9,16.3,22,16,22 s-0.6-0.1-0.8-0.4l-6-8c-0.2-0.3-0.3-0.7-0.1-1S9.6,12,10,12h12c0.4,0,0.7,0.2,0.9,0.6S23,13.3,22.8,13.6z">
    </path>
  </symbol>
</svg>
```
这段代码使用SVG symbols定义了两个图标，每个symbol元素定义一个图标，图标id分别是heart和arrow，将其放在html文件的body元素内。

通过以下代码引用id为heart的图标：
```html
<svg>
    <use xlink:href="#heart"/>
</svg>
```
xlink:href属性值就是‘#’加symbol的id名称，那么只需改变这个属性值就可以引用不同的图标。

## gulp自动化处理

如果你使用gulp构建项目，推荐使用一个专门用于处理SVG Symbols用的glup插件[gulp-svg-symbols](https://github.com/Hiswe/gulp-svg-symbols)，它不但能生成SVG Symbols文件，还能生成一个demo文件方便查看图标和复制代码。

安装gulp-svg-symbols插件，若没有预先安装gulp请先行安装:
```
npm install gulp-svg-symbols  --save-dev
```

gulpfile.js写入如下执行任务：
```js
const gulp = require('gulp')
const svgSymbols = require('gulp-svg-symbols')

gulp.task(`sprites`, function() {
  return gulp
    .src(`assets/svg/*.svg`)
    .pipe(svgSymbols())
    .pipe(gulp.dest(`assets`))
})
```
现在生成SVG symbols文件了，那怎么将它引入到页面呢？如果是多页应用推荐使用[svg4everybody.js](https://www.npmjs.com/package/svg4everybody)为所有浏览器添加了SVG外部内容支持。这样就能直接使用外部的SVG symbols文件。

在文档中包含该脚本：
```html
<script src="/path/to/svg4everybody.js"></script>
<script>svg4everybody(); // run it now or whenever you are ready</script> 
```

使用外部的SVG symbols文件时，通过以下代码引用图标（map.svg是外部的SVG symbols文件，codepen是图标id）：
```html
<svg>
  <use xlink:href="map.svg#codepen"/>
</svg>
```

如果是单页应用，使用svg4everybody.js就感觉太繁琐了，能不能一步搞定啊？当然可以，将将SVG symbols文件转成js文件就好了，推荐使用[gulp-svg-symbols2js](https://www.npmjs.com/package/gulp-svg-symbols2js)将SVG symbols文件转成js文件。

安装gulp-svg-symbols2js插件：
```
npm install gulp-svg-symbols2js --save-dev
```

修改gulpfile.js文件：
```js
const gulp = require('gulp')
const svgSymbols = require('gulp-svg-symbols')
const svgSymbols2js = require('gulp-svg-symbols2js');

gulp.task(`sprites`, function() {
  return gulp
    .src(`assets/svg/*.svg`)
    .pipe(svgSymbols())
    .pipe(svgSymbols2js())
    .pipe(gulp.dest(`assets`))
})
```

现在只要引入生成的js文件，就可以在页面中使用以下代码引用图标：
```html
<svg>
  <use xlink:href="#codepen"/>
</svg>
```

## webpack自动化处理

如果你的项目使用webpack进行打包，可以考虑使用[svg-sprite-loader](https://github.com/kisenka/svg-sprite-loader)插件自动生成SVG symbols文件。

安装svg-sprite-loader:
```
npm install svg-sprite-loader --save-dev
```

将svg图标放到src/icons目录下，在webpack配置文件中添加svg-sprite-loader的配置：
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.svg$/,
        loader: 'svg-sprite-loader',
        include: [path.resolve(__dirname, 'src/icons')], // 仅处理src/icons目录下的svg文件
        options: {
          symbolId: 'icon-[name]'
        }
      }
    ]
  }
}
```

在打包文件（index.js）中引入单个图标：
```js
import cloud from './icons/cloud.svg'
```
这样引入的好处是只引入需要的图标，没引入的图标不会被打包，缺点是当图标很多时这样引入会显得很繁琐，因为一般情况下，所有图标都会被使用，所以有必要找到一种简单的方式一次引入所有图标。

使用一些代码将svg图标全部引入：
```js
const requireAll = requireContext => requireContext.keys().map(requireContext);
const req = require.context('./icons', true, /\.svg$/);
requireAll(req);
```

现在可以在HTML中使用图标了：
```html
<svg>
  <use xlink:href="#icon-cloud" />
</svg>
```

## 经验总结和建议

* 始终使用gulp构建，方便浏览的图标demo在开发中很重要
* 始终使用单色图标，多色图标只能通过id选择器修改样式
* 始终在ie9+，及现代浏览器使用，传统浏览器使用iconfont更好

