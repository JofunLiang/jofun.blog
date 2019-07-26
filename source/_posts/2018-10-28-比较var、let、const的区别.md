---
title: 比较var、let、const的区别
date: 2018-10-28
toc: false
comments: false
tags:
    - 变量提升
    - 块作用域
    - 作用域
categories:
    - JavaScript
---

在前端开发工作中，JavaScript 语言是其核心语言。JavaScript 是一门动态弱类型语言，为什么是动态弱类型语言？这是因为 JavaScript 在声明变量时无需严格指定变量类型，且在变量的使用中可以随时显示或隐式变换类型。因此，理解其变量声明语句是非常基础以及非常重要的。而最常用的声明变量关键字是 var,在ES6版本中新增了let和const声明。

<!--more-->

## var声明与变量提升

### 描述

变量声明，无论发生在何处，都应在执行任何代码之前进行处理。用var声明的变量的作用域是它当前的执行上下文，它可以是嵌套的函数，也可以是声明在任何函数外的变量。如果你重新声明一个 JavaScript 变量，它将不会丢失其值。

将赋值给未声明变量的值在执行赋值时将其隐式地创建为全局变量（它将成为全局对象的属性）。
```js
function foo() {
  a = 1;   // 在严格模式（strict mode）下会抛出ReferenceError异常。
  var b = 2;
}

foo();

console.log(a); // 打印"1" 。
console.log(b); // 抛出ReferenceError: b未在foo外部声明。
```

### 变量提升

由于变量声明（以及其他声明）总是在任意代码执行之前处理的，所以在代码中的任意位置声明变量总是等效于在代码开头声明。这意味着变量可以在声明之前使用，这个行为叫做“hoisting”。“hoisting”就像是把所有的变量声明自动的移动到函数或者全局代码的开头位置。
```js
a = 2
var a;
// ...

// 可以理解为：

var a;
a = 2;
```
因此，建议始终在作用域顶部声明变量（全局代码的顶部和函数代码的顶部），这可以清楚知道哪些变量是函数作用域（本地），哪些变量在作用域链上解决。

变量提升将影响变量声明，但不会影响其值的初始化。当到达赋值语句时，该值依然可以被分配：
```js
function foo() {
  console.log(bar); // undefined，此处bar未声明
  var bar = 111;
  console.log(bar); // 111，此时bar已被声明，并以赋值，值111成功被分配给bar
}

// 相当于: 
function do_something() {
  var bar;
  console.log(bar); // undefined，bar已声明但未分配值。
  bar = 111;
  console.log(bar); // 111
}
```

## let声明与块作用域

### 描述

ES6中引入了块作用域，而let则作为声明块作用域的关键字。let允许你声明一个作用域被限制在块级中的变量、语句或者表达式。与var关键字不同的是，var声明的变量只能是全局或者整个函数块的。

let声明的变量只在其声明的块或子块中可用，这一点，与var相似。二者之间最主要的区别在于var声明的变量的作用域是整个封闭函数。
```js
// var声明
function foo() {
  var x = 1;
  if (true) {
    var x = 2;  // x是同一个变量!
    console.log(x);  // 2
  }
  console.log(x);  // 2
}

// let声明
function foo() {
  let x = 1;
  if (true) {
    let x = 2;  // x是两个不同的变量
    console.log(x);  // 2
  }
  console.log(x);  // 1
}
```

### 用let简化函数代码

当用到内部函数的时候，let能起到简化代码的作用。
```js
var ul = document.getElementById("ul");

for (let i = 1; i <= 5; i++) {
  var item = document.createElement("li");
  item.appendChild(document.createTextNode("Item " + i));

  let j = i;
  item.onclick = function (e) {
    console.log("Item " + j + " is clicked.");
  };
  ul.appendChild(item);
}
```
上面这段代码的作用是为ul元素创建5个li，点击不同的li能够打印出当前li的序号。如果不用let，而改用var的话，将总是打印出 Item 5 is Clicked，因为 j 是函数级变量，5个内部函数都指向了同一个 j ,而 j 最后一次赋值是5。用了let后，j 变成块级域（也就是花括号中的块，每进入一次花括号就生成了一个块级域）,所以 5 个内部函数指向了不同的 j 。在ES6出来之前，由于没有块作用域的概念，因此我们常常用闭包解决，增加了函数的复杂度。

### 用let模仿私有接口

在面向对象编程中处理构造函数的时候，可以通过let声明而不是闭包来创建私有接口。

```js
var Person;
      
{
  let privateScope = {}; // 私有变量
  
  Person = function () {
    this.publicScope = 'foo';
    privateScope.property = 'bar';
  }
  
  Person.prototype.showPublic = function () {
    console.log(this.publicScope);
  }
  
  Person.prototype.showPrivate= function () {
    console.log(privateScope.property);
  }
}

var newPerson = new Person();

newPerson.showPublic();  // foo
newPerson.showPrivate(); // bar

console.log(privateScope.property); // error，privateScope为私用变量，外部无法访问
```

### let对比var

let的作用域是块，而var的作用域是整个函数。
```js
var a = 5;  // 作用域是整个函数内部，整个函数内部都可以访问
var b = 10;

if (a === 5) {
  let a = 4; // 作用域为if块，if块之外无法访问
  var b = 1; // 作用域是整个函数内部

  console.log(a);  // 4，这是if块内的a，if块之外无法访问
  console.log(b);  // 1
} 

console.log(a); // 5, 这是函数内的a，整个函数内部都可以访问
console.log(b); // 1
```

## const声明

### 描述

此声明创建一个常量，其作用域可以是全局或本地声明的块。 与var变量不同，全局常量不会变为窗口对象的属性。需要一个常数的初始化器；也就是说，您必须在声明的同一语句中指定它的值（这是有道理的，因为以后不能更改）。

一个常量不能和它所在作用域内的其他变量或函数拥有相同的名称。
```js
// 定义常量a并赋值7
const a = 7;

// 尝试重新赋值会报错
a = 20;

// 输出 7
console.log("my favorite number is: " + a);

// 尝试重新声明会报错 
const a = 20;
```

const声明创建一个值的只读引用。但这并不意味着它所持有的值是不可变的，只是变量标识符不能重新分配。例如，在引用内容是对象的情况下，这意味着可以改变对象的内容（例如，其参数）。
```javascript
const obj = {
  a: 'foo',
  b: 'bar'
}

obj.a = 'a';
console.log(obj.a) // a，重新赋值成功

obj = {}
console.log(obj) // error，报错
```

## 总结

综上所述，var声明的作用域是整个函数内部，var声明会自动提升到函数最前面；let声明和const声明的作用域都是块；var声明和let声明的变量可以重复赋值，而const声明的是常量，不可以重复赋值，但声明的变量是对象等引用内容的情况下，可以改变对象的内容。
