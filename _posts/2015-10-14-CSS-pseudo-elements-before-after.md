---
layout: post
title: CSS伪元素:before和:after
date: 2015-10-14
tag: [CSS]
category: Frontend
digest: 使用CSS伪元素::before和::after能够充分利用其提供的特性，使得页面看起来更加友好和专业。通常他们被用于添加图标字体和设置或清除页面浮动。本文对其使用进行总结。
---

### :before 和 :after

两者定义**都需要对content进行赋值**，其值可以为字符串，也可以为`url(...)`形式。若没有content属性，则:before或者:after不生效。如果我们不需要生成内容，而又想使:before生效，可以让content值为空字符串：`{ content: ''}`。


    <style>
        /*.before:hover:before{content:'you before'; color:red;}*/
        .before:before{content:'you before'; display: block; color:red;} /* 设置display为block */
        .before:after{content:'him'; color: green;}
    </style>
    
    <div class="before"> me <span> then </span></div>


其结果为:
![结果](http://sfault-image.b0.upaiyun.com/398/648/3986484080-5624ac02a8172_articlex)

由上图可见:before在所修饰元素的开始标签后面，:before所修饰的标签的内容前面。:after在所修饰的标签的内容后面，在所修饰元素的结束标签前面。

上面注释代码可见伪元素可以与伪类一起使用，但是要注意伪元素只能在伪类的后面。

--------

### content取值

参考[CSS3的content属性详解](http://segmentfault.com/a/1190000002750033)。常见的如字符串，图片，标点符号，元素属性值和项目编号。

content设置为元素属性值，即配合取值函数 attr() 一起使用，如：

    a::before{content: attr(title)}
    <a href="http://www.segmentfault.com" title="专业面向开发者的中文技术问答社区"></a>

此时达到的效果相当于：

    <a href="http://www.segmentfault.com">专业面向开发者的中文技术问答社区</a>

--------

### :before和:after设置图标字体

通常是在:before中设置图标字体。可以使用[Fontello](http://fontello.com/)或者[Font Awesome](http://fontawesome.io)。

使用Fontello需要自选图标字体然后下载，较麻烦，但是集成其他网站的图标，没有被墙。后者图标较多，使用方便，直接引入字体css文件即可使用，但是下载字体css文件由于国内环境需要翻墙。

Font Awesome提供很多额外功能，如按大小获得图标、对对齐要求严格的地方使用fa-fw设置固定宽度、在列表中作为列表项的图标、设置图标边界fa-border和环绕方式fa-pull-left或fa-pull-right、设置图标动画fa-spin和fa-pulse、设置图标旋转fa-rotate和翻转fa-flip，以及非常有用的组合不同图标fa-stack——这通常组合一大一小图标，所以会使用fa-stack-2x获得一个2倍大的背景图标和一个正常大小的图标作为内容，使用fa-inverse使两个图标颜色形成反色，突出组合后的图标内容。此外，Font Awesome还可以同Bootstrap3完美接合使用。

此外，国内[阿里巴巴字体库](http://www.iconfont.cn)内容多，不会被墙，也有很多功能。

--------

##Todo

`:before和:after还有一种常用方式是content值为空或空格，通过border相关属性设置微型图形，
如三角形或圆角等，需进一步学习。`
    
[用CSS绘制三角形](http://segmentfault.com/a/1190000002783179)

[css使用border画三角形](http://segmentfault.com/a/1190000003833676)

[css border](http://segmentfault.com/search?q=css+border+)
    
`:before和:after也可用于清除浮动。`
    


--------

### 参考资料
[详解 CSS 属性 - :before && :after](http://segmentfault.com/a/1190000000474414)

