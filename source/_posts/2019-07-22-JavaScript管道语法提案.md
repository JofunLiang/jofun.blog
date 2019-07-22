---
title: JavaScript管道语法提案
date: 2019-07-22
toc: false
comment: true
tags:
    - 管道语法
    - 管道函数
categories:
    - JavaScript
---

管道语法引入了一个 |> 类似于 F＃， OCaml， Elixir， Elm， Julia， Hack和LiveScript的新运算符，以及UNIX管道。它是一种向后兼容的方式，以可读，功能的方式简化链式函数调用，并提供了扩展内置原型的实用替代方法。

<!--more-->

## 单个参数的函数调用

对于具有单个参数的函数调用，管道运算符本质上是一个有用的语法糖。换句话说，sqrt(64)相当于64 |> sqrt。

当将多个功能链接在一起时，这具有更好的可读性。例如，给定以下功能：
```js
function doubleSay (str) {
  return str + ", " + str;
}

function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}

function exclaim (str) {
  return str + '!';
}
```

...以下调用是等效的：
```js
let result = exclaim(capitalize(doubleSay('hello')));
// result => 'Hello, hello!'

let result = 'hello'
  |> doubleSay
  |> capitalize
  |> exclaim;
// result => 'Hello, hello!'
```

## 具有多个参数的函数

对于具有多个参数的函数，管道运算符不需要任何特殊规则; JavaScript已经有办法处理这种情况。

例如，给定以下功能：
```js
function double (x) {
  return x + x;
}

function add (x, y) {
  return x + y;
}

function boundScore (min, max, score) {
  return Math.max(min, Math.min(max, score));
}
```
...您可以使用箭头函数来处理多参数函数（例如add）：
```js
let person = {
  score: 25
}

let newScore = person.score
  |> double
  |> (_ => add(7, _))
  |> (_ => boundScore(0, 100, _));
// newScore => 57
```
相当于:
```js
let newScore = boundScore( 0, 100, add(7, double(person.score)) );
// newScore => 57
```
注意：_不需要使用下划线; 它只是一个箭头功能，所以你可以使用你喜欢的任何参数名称。

正如您所看到的，因为管道运算符总是管道单个结果值，所以它与单参数箭头函数语法非常相似。此外，由于管道运算符的语义是纯粹而简单的，因此JavaScript引擎可以优化箭头函数。

## 构建工具

Babel管道支持插件：[@babel/plugin-proposal-pipeline-operator](https://babeljs.io/docs/en/next/babel-plugin-proposal-pipeline-operator.html)

安装：
```
$ npm install @babel/plugin-proposal-pipeline-operator
```

配置.babelrc插件选项：
```js
{
  "plugins": [
    [
      "@babel/plugin-proposal-pipeline-operator",
      {
        "proposal": "minimal"
      }
    ]
  ]
}
```

## 附录
[管道语法提案](https://github.com/tc39/proposal-pipeline-operator)