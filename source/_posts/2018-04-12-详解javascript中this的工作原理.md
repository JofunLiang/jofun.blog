---
title: 详解javascript中this的工作原理
date: 2018-04-12
toc: false
comments: false
tags:
    - this的工作原理
categories:
    - JavaScript
---

在 JavaScript 中 this 常常指向方法调用的对象，但有些时候并不是这样的，本文将详细解读在不同的情况下 this 的指向。

<!--more-->

## 指向 window

在全局中使用 this，它将会指向全局对象，因为浏览器中运行的 JavaScript 的全局对象默认为 window，所以，此时 this 指向 window：
```js
console.log(this) // 控制台将打印出 window 对象
```
在全局作用域内的函数调用， this 也会执行 window：
```js
function foo(){
  console.log(this);
};

foo(); // 控制台也会打印出 window 对象
```
此处并不难理解，因为全局对象默认为 window，则 foo() 实质是 window.foo()。

其实，在JavaScript中函数调用时，this都会指向window对象。看下面的执行结果：
```js
function foo(){
  console.log(this);
};
            
var demo = document.querySelector(".demo");
            
demo.onclick = foo;         //this指向demo元素对象
            
demo.onclick = function(){
  foo();                  //this指向window对象
};
```
*注意：在 ES5 中，使用严格模式时，不存在全局变量， 此时 this 将不再指向 window， 而是 undefined 。*

## 指向方法调用的对象

在对象的方法调用中，this 指向该方法调用的对象：
```js
var obj = {
  getMe: function(){
    console.log(this)
  }
};

obj.getMe(); // 控制台打印出 obj 对象
```

## 构造函数内，指向调用构造函数所创建的对象实例

通常我们会使用 new 关键词调用构造函数创建新的对象实例，此时构造函数内的 this 就会指向这个新创建出来的对象：
```js
//构造函数
function Person(name){
  this.name = name;
  this.getMe = function(){
    console.log(this);
  }
};

var joe = new Person("joe");

joe.getMe(); //控制台打印出一个新的名字为 “joe” 的 Person 实例
```

## 使用函数的 call、apply 或 bind 方法时，this 将会被显式设置为函数调用的第一个参数

使用函数的 call、apply 或 bind 方法时，this 将会被显式设置为函数调用的第一个参数：
```js
var obj = {
  name: "object"
};

function a(){
  console.log(this)
};

a.call(obj); //控制台打印出 obj 对象
```
出现这样的结果是由 call、apply 和 bind 的实现原理决定的，call、apply 和 bind 改变 this 指向的原理是它动态改变了函数的运行上下文环境。