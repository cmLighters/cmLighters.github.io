---
layout: post
title: CSS 选择器
date: 2015-05-25
meta: CSS
digest: CSS选择器在前端编程中占有重要位置。通过选择器对不同元素定义不同的样式，也可以通过选择器编写JavaScript代码修改选择器定义的元素的样式和行为。因此理清选择器的功能非常重要。由于其种类多，造成不同选择器功能容易混淆。从CSS1定义最基本最常用的选择器到CSS3中不断补充其他选择器（主要是完善伪类选择器）。将网上搜集的资料和自己所学记下来，待日后快速复查。
tags: [CSS]
category: [Frontend]
---

CSS选择器在前端代码编写中占有重要位置。通过选择器可以选择不同元素定义不同的样式，也可以通过选择器编写JavaScript代码修改所选元素的样式和行为。因此理清选择器的功能非常重要。由于其种类多，造成不同选择器功能容易混淆。从CSS1定义最基本最常用的选择器到CSS3中不断补充其他选择器（主要是完善伪类选择器）。将自己学习中觉得重要和容易混淆的地方记录下来，待日后需要时能够快速复查。

选择器总共有四大类： 

 - 基本选择器
 - 组合选择器
 - 伪类
 - 伪元素

在基本选择器和组合选择器中还分不同种类

------------

## 基本选择器
基本选择器下有一下几种：

 - element（标签选择器）
 - .class
 - \#id
 - *
 - 属性选择器
    + [attr]
    + [attr=value]
    + [attr~=value]
    + [attr\|=value]
    + [attr^=value]
    + [attr$=value]
    + [attr\*=value]

**【注意】**  
上述[attr=value]选择的是attr属性值只为value的而不是包含value单词的字串，如“value othervalue”。[attr|=value]和[attr^=value]的区别在于前者attr的值必须为value或者/$value-.\*/；后者要求值为/value.\*/即可。[attr~=value]和[attr*=value]的区别在于前者attr值中有一个值为value（即attr的值由多个子值构成，这些子值由空格分隔，其中有一个子值是value），后者要求attr的值字符串中包含value子串。
            
--------

##组合选择器
组合选择器有如下几种：

 - selector1, selector2   多元素选择器
 - former\_element + target\_element    相邻兄弟选择器
 - element1 ~ element2      普通兄弟选择器， 选择p之后的所有element2兄弟
 - selector1 > selector2    子元素选择器
 - selector1 selector2      后代选择器

--------

## 伪元素选择器
有如下几种：

 - :after
 - :before
 - :first-letter
 - :first-line
 - ::selection

伪元素较常用的是[`:before`和`:after`](http://segmentfault.com/a/1190000000474414)。

::selection只能以“::”形式定义，对光标或鼠标移动选中的区域样式进行定义。现阶段只能修改 `color`，`background-color`， `cursor`， `outline`， `text-decoration`， `text-emphasis-color`和`text-shadow` 等特性。其他的包括`background-image`会被忽略。

伪元素种类少，不易混淆。新增加的伪元素::backdrop属于实验特性，先有个了解就可以了。


--------

## 伪类选择器

有如下多种：

 - 链接伪类
    - :link
    - :visited
    - :active
    - :hover
 - 结构伪类
    - first-child   在CSS2中定义，其他结构性伪类在CSS3中定义
    - last-child
    - only-child
    - nth-child(n)
    - nth-last-child(n)
    - first-of-type
    - last-of-type
    - only-of-type
    - nth-of-type(n)
    - nth-last-of-type(n)
    - :root
    - :empty
 - 界面相关伪类
    - :enabled
    - :disabled
    - :checked
    - :optional
    - :required
    - :read-write
    - :read-only
 - 值相关伪类
    - :in-range
    - :out-of-range
    - :valid
    - :invalid
 - :not(selector)     匹配不符合当前选择器的任何元素
 - :target    匹配文档中特定"id"点击后的效果

:target伪类用于当点击`<a href="#value">...</a>`标签的元素时，对应id为value的页面样式显示为target现在的css值的显示效果。


--------    

## 参考资料

 1. [阮一峰CSS选择器笔记](http://www.ruanyifeng.com/blog/2009/03/css_selectors.html)
 2. [证服高级CSS选择器](http://www.qianduan.net/taming-advanced-css-selectors.html)
 3. [牢记31种CSS选择器用法](http://peiwen.lu/css-selectors-must-memorize/)
 4. [CSS 3.0 参考手册](http://www.qianduan.net/recommended-css-3-0-reference-manual.html)
 5. [CSS 选择器列表](http://www.runoob.com/cssref/css-selectors.html)
 6. [MDN CSS Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)

--------
