---
title: 设置placeholder样式及隐藏IE浏览器input元素的删除和查看密码图标
date: 2018-04-23
toc: false
comment: true
tags:
    - 设置placeholder样式
    - 设置input样式
categories:
    - CSS
---

如何设置placeholder样式？IE浏览器input元素的删除和查看密码图标？

<!--more-->

## 设置placeholder样式

在做项目的时候，一般表单元素的placeholder属性样式都是使用浏览器默认的，但有时候为了追求设计上的美感需要修表单元素的placeholder样式（也有可能是遇到了一个处女座的设计师或者是客户），就不等不修改一下placeholder的样式。可以通过下面的代码修改样式：
```css
/*Chrome、Safari等 webkit内核浏览器*/
::-webkit-input-placeholder{
  color:red;
}
            
/*Firefox*/
::-moz-placeholder{
  color:red;
}
            
/*IE、Edge等 Trident 内核浏览器*/
:-ms-input-placeholder{
  color:red;
}
```

## 隐藏IE浏览器input元素的删除和查看密码图标

在IE、Edge等 Trident 内核浏览器中，type = “text” 的 input元素中有输入时会出现清除图标，type = “password” 的 input元素中有输入时会出现眼睛图标。添加下面的样式可以去除默认图标：
```css
::-ms-clear, 
::-ms-reveal{
  display: none;
}
```