---
title: javaScript识别网址并转为链接
date: 2018-04-08
toc: false
comment: true
tags:
    - JavaScript
categories:
    - JavaScript
---

最近项目有个需求：用户发送消息时，如果发送者输入的信息中含有网址文本，要在接受者界面中显示网址链接，点击该链接直接跳转到网页。这个功能和 QQ 发送网址文本的效果类似。

<!--more-->

## 思路

首先，要判断文本中是否含有网址文本，其次，将网址文本转换为可点击的链接文本，即将网址文本通过a标签括起来。

## 实现过程

### 识别网址

在 javaScript 中判断某种特殊格式的文本，首选正则表达式，下面是我用来检查网址的正则：
```js
var re = /^(f|ht){1}(tp|tps):\\/\\/([\\w-]+\\.)+[\\w-]+(\\/[\\w- ./?%&=]*)?/g;
```

这里需要注意的是，正则必须使用全局匹配 g 。否则只能匹配到文本中的第一个网址文本。

### 网址转换为链接

在网址转换中涉及字符串的操作，那么自然要使用 String 对象的方法，先复习下 String 对象能与正则表达式一起使用的方法有哪些？

常用的有这几个：

search：检索与正则表达式相匹配的值。
match：找到一个或多个正则表达式的匹配。
replace：替换与正则表达式匹配的子串。
split：把字符串分割为字符串数组。

可以看出来，其中 replace 是最方便、最适合这个需求的。

### replace函数的用法

语法：
```
string.replace(searchvalue,newvalue)
```
参数解析：

searchvalue：必须。规定子字符串或要替换的模式的 RegExp 对象。请注意，如果该值是一个字符串，则将它作为要检索的直接量文本模式，而不是首先被转换为 RegExp 对象。

newvalue：必需。一个字符串值。规定了替换文本或生成替换文本的函数。

注意：第二个参数支持使用函数来制定文本替换的规则。

回顾需求，要将网址转换为a链接,那么得到的转换规则如下：
```html
url => <a href='url' target='_blank'>url</a>
```

## 封装实现函数

根据上面的分析过程，使用代码来描述如下：
```js
var urlToLink = function(str){
    var re = /^(f|ht){1}(tp|tps):\\/\\/([\\w-]+\\.)+[\\w-]+(\\/[\\w- ./?%&=]*)?/g; 

    str = str.replace(re, function(website){ 
        return "<a href='" + website +"' target='_blank'>" + website + "</a>"; 
    }); 
    return str;
};
```
到这里，javaScript识别网址文本并转为链接文本的函数接完成了。
