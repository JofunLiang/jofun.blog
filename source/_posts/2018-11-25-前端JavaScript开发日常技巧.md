---
title: 前端JavaScript开发日常技巧
date: 2018-11-25
toc: false
comments: false
tags:
    - JavaScript技巧
categories:
    - JavaScript
---

如果你是一个JavaScript新手或仅仅最近才在你的开发工作中接触它，你可能感到失望。所有的语言都有自己的怪癖————JavaScript也不例外，与其他语言相比JavaScript甚至是有过之而无不及。在这篇文章中，我将分享一些JavaScript的怪异行为和一些最常用的技巧，希望我能分享给你一些曾经令我头痛不已的经验。这不是一个完整列表——仅仅是一部分————但希望它让你看清这门语言的强大之处，可能曾经被你认为是障碍或者是你正在困惑的东西。

<!--more-->

## 真（true）和假（false)

JavaScript并没有一开始就告诉你什么值被认为是真（true）或假（false)，这就是我曾经令我很头痛和困惑的事情。在开发中经常会遇到NaN、null、undefined这样的值，我不得不先对他们进行真（true）/假（false）的判断，然后再编写我需要处理的程序逻辑。但现在我已经能够轻松的觉解这类问题，因为我发现在JavaScript中被判定为假（false）的值只有一下几种：

```javascript

const a = Boolean('');           // false
const b = Boolean(NaN);          // false
const c = Boolean(0);            // false
const d = Boolean(null);         // false
const e = Boolean(undefined);    // false
const f = Boolean(false);        // false

```

你完全可以测试他们，是否如我所说的那样会被认为是假（false），下面是一个测试函数：

```javascript

(function test () {
  var array = [false, '', NaN, 0, null, undefined];
  array.forEach(value => {
    if (value) {
      console.log(true);
    } else {
      console.log(false);
    }
  })
})();

```

现在你只需要记住以上的几个为假（false）的值，其他的都为真（true）。

## 相等判断

在大多数语言里==比较运算符————值类型（或字符串）当有相同值是是相等的。引用类型相等需要有相同的引用。刚开始的我很惊讶为什么JavaScript有两个等值运算符：==和===。最初我的大部分代码都是用的==，所以我并不知道当我运行如下代码的时候JavaScript为我做了什么：

```javascript

const a = 1;

console.log(a == '1' ? true : false);   // true

```

最后的结果是真（true），这就是让我困惑的地方————数字1和字符串‘1’怎么会是相等的？

在JavaScript中，有相等（==）和严格相等（===）之说。（==）运算符将强制转换两边的操作数为相同类型后执行严格相等比较。所以在上面的例子中，字符串‘1’会被转换为整数1，这个过程在幕后进行，然后与变量a进行比较。

严格相等（===）不进行类型转换。如果操作数类型不同（如整数和字符串），那么他们不全等（严格相等）：

```javascript

const a = 1;

console.log(a === '1' ? true : false);   // false

```

你可能正在联想可能发生强制类型转换而引起的各种恐怖问题————假设你的引用中发生了这种转换，可能导致你非常困难找到问题出在哪里。这并不奇怪，这也是为什么经验丰富的JavaScript开发者总是建议使用严格相等（===）。所以我总是使用（===）判断两个值是否相等。

## 三元表达式

合理的使用三元表达式可以让你的代码看起来更简洁。在我的工作中，我通常使用三元表达式来消除一般的if...else语句。这样能使我的代码看起来更简洁，书写更高效。
举个例子，大于18岁的人为成年人，小于18岁的为未成年人：

```javascript
const age = 21;
var isAdults;

// 使用if...else语句
if (age >= 18) {
  isAdults = true
} else {
  isAdults = false
}

// 使用三元表达式
isAdults = age >= 18
```
很明显，上面的例子中三元表达式比if...else语句要简洁的多。

我们还可以将例子的复杂度提高————大于40岁的称为中年人：

```javascript
const age = 21;
var people;

// 使用if...else语句
if (age < 18) {
  people = '未成年人';
} else if (age < 40) {
  people = '成年人';
} else {
  people = '中年人';
}

// 使用三元表达式
people = age < 18 ? '未成年人' : age < 40 ? '成年人' : '中年人'
```

## 使用 try...catch... 语句屏蔽错误

频繁的报错会使网站的性能低下，影响用户体验。一般的小项目站点有一两处报错可能影响不大，可是一旦项目体量变大了呢，这时如果出现了站点报错，你应该去重视它并且解决它。在我的经验中，我能通常使用 try...catch... 语句屏蔽一些我不希望出现的错误。

比如，我需要使用数组的map方法处理数组，因为我无法保证arr是不是数组，所以我使用 try...catch... 语句，如果arr不是数组会执行catch分支返回一个空数组。这样，代码就不会报错还能保证最后返回的一定是数组：

```javascript
try {
  return arr.map(item => {
    // do something
  })
} catch (e) {
  return []
}
```
try...catch... 语句是一把双刃剑，千万不要滥用它。滥用它的结果那就是————你的程序执行异常，但你却找不到错误发生在哪里。