---
title: Vue组件样式scoped的原理与样式穿透的用法
date: 2018-09-13
toc: false
comment: true
tags:
    - vue组件样式
    - scoped样式
    - PostCss
categories:
    - Vue
---

在Vue文件中的style标签上有一个特殊的属性——scoped。当一个style标签拥有scoped属性时候，它的css样式只能作用于当前的Vue组件。有了这个属性，可以使Vue组件的样式相互不被污染。这就相当于实现了样式的模块化。

<!--more-->

## scoped的实现原理

Vue中的scoped属性的效果主要是通过PostCss实现的。

转译前的代码：
```html
<style scoped>
    .example{
        color:red;
    }
</style>
<template>
    <div class="example">scoped属性例子</div>
</template>
```

转译后的代码：
```html
.example[data-v-5558831a] {
  color: red;
}
<template>
    <div class="example" data-v-5558831a>scoped属性例子</div>
</template>
```

PostCSS给一个组件中样式对应的dom添加了一个独一无二的动态属性，给css选择器额外添加一个对应的属性选择器，来选择组件中的dom,这种做法使得样式只作用于含有该属性的dom元素(组件内部的dom)。

scoped的渲染规则：

1、给HTML的dom节点添加一个不重复的data属性(例如: data-v-5558831a)来唯一标识这个dom 元素。

2、在每个css选择器的末尾(编译后生成的css语句)加一个当前组件的data属性选择器(例如：[data-v-5558831a])来私有化样式。

## scoped样式穿透

scoped在Vue组件中实现了样式的模块化，但同时也带来了一些问题——修改全局样式。当时在Vue项目中，当我们引入第三方组件库时，需要在局部组件中修改第三方组件库的样式，而又不想去除scoped属性造成组件之间的样式覆盖。这时我们可以通过特殊的方式穿透scoped。

stylus预处理语言的样式穿透， 使用：>>>

例如：
```css
.wrapper >>> .el-button {
	border-radius:0;
}
```

sass和less预处理语言的样式穿透，使用：/deep/

例如：
```css
.wrapper /deep/ .el-button {
	border-radius:0;
}
```

scoped穿透规则：外层 穿透符号 第三方组件样式/全局样式

## 样式穿透的其它方法

1、在vue组建中使用两个style标签，一个加上scoped属性，一个不加scoped属性，把需要覆盖的css样式写在不加scoped属性的style标签里。

2、建立一个reset.css(基础全局样式)文件，里面写覆盖的css样式，在入口文件main.js 中引入。





