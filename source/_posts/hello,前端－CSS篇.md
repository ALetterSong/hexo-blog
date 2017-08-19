---
title: hello,前端－CSS篇
date: 2016-08-18 17:10:24
tags:
---



> hello,前端－HTML篇  
**hello,前端－CSS篇**
...  
本文大部分内容总结自[w3school](http://www.w3school.com.cn/css/index.asp)和[w3help](http://w3help.org/zh-cn/kb/)。
本文将使用CSS3的新特性。  
[CSS之BFC](http://www.jianshu.com/p/ca8ba6464a61)  
[CSS包含块](http://www.jianshu.com/p/f449f37b63f8)  
[CSS清除浮动](http://www.jianshu.com/p/b4fd2f5d1844)  

CSS 是 Cascading Style Sheet（层叠样式表）的缩写。是用于（增强）控制网页样式并允许将样式信息与网页内容分离的一种标记性语言。  
CSS 不需要编译,可以直接由浏览器执行(属于浏览器解释型语言)。


### 语法

CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明。  
选择器通常是您需要改变样式的 HTML 元素。  
每条声明由一个属性和一个值组成。  
属性（property）是您希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开。

![CSS结构](http://www.w3school.com.cn/i/ct_css_selector.gif)

### 创建CSS

插入样式表的方法有三种:  

外部样式表

	<!-- Use link elements -->
	<link rel="stylesheet" href="core.css">
	
	<!-- Use @imports -->
	<style>
	  @import url("more.css");
	</style>
		
*推荐使用 HTML 的 link 对象来引入参见 [http://zoomzhao.github.io/code-guide/#css-import](http://zoomzhao.github.io/code-guide/#css-import)*

内部样式表

	<head>
	<style type="text/css">
	  hr {color: sienna;}
	  p {margin-left: 20px;}
	  body {background-image: url("images/back40.gif");}
	</style>
	</head>

内联样式

	<p style="color: sienna; margin-left: 20px">
	This is a paragraph
	</p>
	
### 选择器权重
![选择器权重](http://7xq3d5.com1.z0.glb.clouddn.com/%E9%80%89%E6%8B%A9%E5%99%A8%E6%9D%83%E9%87%8D.jpg)

遵循如下法则：  

- 选择器都有一个权值，权值越大越优先；
- 当权值相等时，后出现的样式表设置要优于先出现的样式表设置；
- 创作者的规则高于浏览者：即网页编写者设置的 CSS 样式的优先权高于浏览器所设置的样式；
- 继承的 CSS 样式不如后来指定的 CSS 样式；
- 在同一组属性设置中标有`!important`规则的优先级最大

### 框模型
CSS 框模型 (Box Model) 规定了元素框处理元素内容、内边距、边框 和 外边距 的方式。

元素框的最内部分是实际的内容，直接包围内容的是内边距。内边距呈现了元素的背景。内边距的边缘是边框。边框以外是外边距，外边距默认是透明的，因此不会遮挡其后的任何元素。

![框模型](http://www.w3school.com.cn/i/ct_boxmodel.gif)

[CSS边框属性](http://www.w3school.com.cn/css/css_border.asp)
[KB006: CSS 框模型( Box module )](http://w3help.org/zh-cn/kb/006/)

### 外边距合并

外边距合并指的是，当两个垂直外边距相遇时，它们将形成一个外边距。
合并后的外边距的高度等于两个发生合并的外边距的高度中的较大者。

![外边距合并](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_1.gif)

![包含元素外边距合并](http://www.w3school.com.cn/i/ct_css_margin_collapsing_example_2.gif)

**注释**：只有普通文档流中块框的垂直外边距才会发生外边距合并。行内框、浮动框或绝对定位之间的外边距不会合并。

**1.两个或多个毗邻的普通流中的块元素垂直方向上的 margin 会折叠**  
关键词：  
**两个或多个** 

说明其数量必须是大于一个，又说明，折叠是元素与元素间相互的行为，不存在 A 和 B 折叠，B 没有和 A 折叠的现象。

**毗邻**  
是指没有被非空内容、`padding`、`border` 或 `clear` 分隔开，说明其位置关系。  

注意一点，在没有被分隔开的情况下，一个元素的 `margin-top` 会和它普通流中的第一个子元素(非浮动元素等)的 `margin-top` 相邻； 只有在一个元素的 `height` 是 "auto" 的情况下，它的 `margin-bottom` 才会和它普通流中的最后一个子元素(非浮动元素等)的 `margin-bottom` 相邻。

**垂直方向**  
是指具体的方位，只有垂直方向的 margin 才会折叠，也就是说，水平方向的 margin 不会发生折叠的现象。  

**2.浮动元素、inline-block 元素、绝对定位元素的 margin 不会和垂直方向上其他元素的 margin 折叠**

**3.创建了块级格式化上下文的元素，不和它的子元素发生 margin 折叠**

**4.元素自身的 margin-bottom 和 margin-top 相邻时也会折叠**

### 定位机制
CSS 定位 (Positioning) 属性允许你对元素进行定位。  

CSS 有三种基本的定位机制：**普通流**、**浮动**和**绝对定位**。
除非专门指定，否则所有框都在普通流中定位。也就是说，普通流中的元素的位置由元素在 HTML 中的位置决定。
块级框从上到下一个接一个地排列，框之间的垂直距离是由框的垂直外边距计算出来。
行内框在一行中水平布置。可以使用水平内边距、边框和外边距调整它们的间距。但是，垂直内边距、边框和外边距不影响行内框的高度。由一行形成的水平框称为行框（Line Box），行框的高度总是足以容纳它包含的所有行内框。不过，设置行高可以增加这个框的高度。

[KB009: CSS 定位体系概述](http://w3help.org/zh-cn/kb/009/)

***CSS position 属性* ** 

**static**
元素框正常生成。块级元素生成一个矩形框，作为文档流的一部分，行内元素则会创建一个或多个行框，置于其父元素中。
**relative**
元素框偏移某个距离。元素仍保持其未定位前的形状，它原本所占的空间仍保留。
**absolute**
元素框从文档流完全删除，并相对于其包含块定位。包含块可能是文档中的另一个元素或者是初始包含块。元素原先在正常文档流中所占的空间会关闭，就好像元素原来不存在一样。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。
**fixed**
元素框的表现类似于将 position 设置为 absolute，不过其包含块是视窗本身。
提示：相对定位实际上被看作普通流定位模型的一部分，因为元素的位置相对于它在普通流中的位置。

### 包含块

在 CSS2.1 中，很多框的定位和尺寸的计算，都取决于一个矩形的边界，这个矩形，被称作是包含块( containing block )。 
一般来说，(元素)生成的框会扮演它子孙元素包含块的角色；我们称之为：一个(元素的)框为它的子孙节点建造了包含块。包含块是一个相对的概念。

![包含块](http://w3help.org/zh-cn/kb/008/008/CB4.png)

**注意**：*绝对定位（"position: absolute"）*元素的包含块由离它最近的 'position' 属性为 'absolute'、'relative' 或者 'fixed' 的祖先元素创建。如果其祖先元素是行内元素，则包含块取决于其祖先元素的 'direction' 特性。如果祖先元素不是行内元素，那么包含块的区域应该是祖先元素的内边距边界。
**注意**：*视口*，即可视窗口。当可视窗口的尺寸大小改变时，浏览器应该改变文档的布局。 比如，有些值依赖于可视窗口的大小，DIV 'width' 的 "auto" 值，等等。当可视窗口的尺寸小于文档渲染的画布（也就是页面）的大小时，浏览器应该提供滚动机制。 每个画布最多有一个可视窗口。但是，浏览器可以同时渲染多个画布。

[KB007: 可视化格式模型( visual formatting model )](http://w3help.org/zh-cn/kb/007/)
[KB008: 包含块( Containing block )](http://w3help.org/zh-cn/kb/008/)

### 相对定位

设置为相对定位的元素框会偏移某个距离。元素仍然保持其未定位前的形状，它原本所占的空间仍保留。

    #box_relative {
      position: relative;
      left: 30px;
      top: 20px;
    }

![相对定位](http://www.w3school.com.cn/i/ct_css_positioning_relative_example.gif)

**注意**：在使用相对定位时，无论是否进行移动，元素仍然占据原来的空间。因此，移动元素会导致它覆盖其它框。

### 绝对定位

设置为绝对定位的元素框从文档流完全删除，并相对于其包含块定位，包含块可能是文档中的另一个元素或者是初始包含块。元素原先在正常文档流中所占的空间会关闭，就好像该元素原来不存在一样。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。

    #box_absolute {
      position: absolute;
      left: 30px;
      top: 20px;
    }
 
![绝对定位](http://www.w3school.com.cn/i/ct_css_positioning_absolute_example.gif)

绝对定位的元素的位置相对于最近的已定位祖先元素，如果元素没有已定位的祖先元素，那么它的位置相对于最初的包含块。

### 浮动

浮动的框可以向左或向右移动，直到它的外边缘碰到包含框或另一个浮动框的边框为止。
由于浮动框不在文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样。

**清除（闭合）浮动**
使用 clear 样式清除

    .clear-float {clear:both;} 

使用伪元素 :after 清除

    .after-clear-float :after{content:””; display:block; clear:both;}

使用 overflow 清除

    .overflow-clear-float {overflow:hidden;}
    或者
    .overflow-clear-float {overflow:auto;}
 
overflow 样式值为 非 visilbe 时，实际上是创建了 CSS 2.1 规范定义的 Block Formatting Contexts。创建了它的元素，会重新计算其内部元素位置，从而获得确切高度。这样父容器也就包含了浮动元素高度。这个名词过于晦涩，在 CSS 3 草案中被变更为名词 Root Flow，顾名思义，是创建了一个新的根布局流，这个布局流是独立的，不影响其外部元素的。实际上，这个特性与 早期 IE 的 hasLayout 特性十分相似。

使用 display:table 清除

    .table-clear-float {display:table}
    或者
    .table-clear-float {display:table-cell}

当元素 display 值被设定为 table 或 table-cell 时，同样也创建了 CSS 2.1 规范定义的 Block Formatting Contexts。这样父容器也就包含了浮动元素高度。

使用表格类元素作为浮动元素容器
使用浮动父元素清除
触发 hasLayout 清除

### 选择器

**元素选择器**

    html {color:black;}
    h1 {color:blue;}
    h2 {color:silver;}
 
**选择器分组**

    //将任意多个选择器分组在一起
    body, h2, p, table, th, td, pre, strong, em { color: gray; }
    //通配符选择器
    * { color: red; }
    //声明分组
    h1 { font: 28 px Verdana;color: white;background: black; }

**类选择器**

    .important {font-weight:bold;}
    .warning {font-style:italic;}
    .important.warning {background:silver;}

**ID选择器**

    #intro {font-weight:bold;}

类选择器还是 ID 选择器？
在类选择器这一章中我们曾讲解过，可以为任意多个元素指定类。前一章中类名 important 被应用到 p 和 h1 元素，而且它还可以应用到更多元素。
**区别 1：只能在文档中使用一次**
与类不同，在一个 HTML 文档中，ID 选择器会使用一次，而且仅一次。
**区别 2：不能使用 ID 词列表（多个词）**
不同于类选择器，ID 选择器不能结合使用，因为 ID 属性不允许有以空格分隔的词列表（多个词）。
**区别 3：ID 能包含更多含义**
类似于类，可以独立于元素来选择 ID。例如，您可能知道在一个给定的文档中会有一个 ID 值为 mostImportant 的元素。您不知道这个最重要的东西是一个段落、一个短语、一个列表项还是一个小节标题。您只知道每个文档都会有这么一个最重要的内容，它可能在任何元素中，而且只能出现一个。

**属性选择器**

属性选择器可以根据元素的属性及属性值来选择元素。

    //把包含标题（ title） 的所有元素变为红色
    [title] { color: red; }
    
    //将同时有 href 和 title 属性的 HTML 超链接的文本设置为红色
    a[href][title] {color:red;}
    
    //将指向 Web 服务器上某个指定文档的超链接变成红色
    a[href="http://www.w3school.com.cn/about_us.asp"] {color: red;}

如果需要根据属性值中的词列表的某个词进行选择，则需要使用波浪号（~）。

    p[class~="important"] {color: red;}

如果忽略了波浪号，则说明需要完成完全值匹配。

*子串匹配属性选择器*

`[abc^="def"]`	选择 abc 属性值以 "def" 开头的所有元素
`[abc$="def"]`	选择 abc 属性值以 "def" 结尾的所有元素
`[abc*="def"]`	选择 abc 属性值中包含子串 "def" 的所有元素

如果希望对指向 W3School 的所有链接应用样式，不必为所有这些链接指定 class，再根据这个类编写样式，而只需编写以下规则：

    a[href*="w3school.com.cn"] {color: red;}

*特定属性选择类型*
        
    //选择 lang 属性等于 en 或以 en- 开头的所有元素
    *[lang|="en"] {color: red;}

**后代选择器**

后代选择器（descendant selector）又称为包含选择器。
后代选择器可以选择作为某元素后代的元素。

    //只对 h1 元素中的 em 元素应用样式
    h1 em { color: red; }

**子元素选择器**

与后代选择器相比，子元素选择器（Child selectors）只能选择作为某元素子元素的元素。

    //选择只作为 h1 元素子元素的 strong 元素
    h1 > strong {color:red;}

**相邻兄弟选择器**

相邻兄弟选择器（Adjacent sibling selector）可选择紧接在另一元素后的元素，且二者有相同父元素。

    //增加紧接在 h1 元素后出现的段落的上边距
    h1 + p {margin-top:50px;}

这个选择器读作：“选择紧接在 h1 元素后出现的段落，h1 和 p 元素拥有共同的父元素”。

**伪类**

CSS 伪类用于向某些选择器添加特殊的效果。

`:active`	向被激活的元素添加样式。	
`:focus`	向拥有键盘输入焦点的元素添加样式。	
`:hover` 当鼠标悬浮在元素上方时，向元素添加样式。	
`:link`	向未被访问的链接添加样式。	
`:visited`	向已被访问的链接添加样式。	
`:first-child`	向元素的第一个子元素添加样式。	
`:lang`	向带有指定 lang 属性的元素添加样式。	

**注意**：伪类很容易遭到误解，查看[在线实例](http://www.w3school.com.cn/css/css_pseudo_classes.asp)

**伪元素**

CSS 伪元素用于向某些选择器设置特殊效果。

`:first-letter`	向文本的第一个字母添加特殊样式。
`:first-line`	向文本的首行添加特殊样式。	
`:before`	在元素之前添加内容。	
`:after`	在元素之后添加内容。	


### 样式

**文本**

通过文本属性，您可以改变文本的颜色、字符间距，对齐文本，装饰文本，对文本进行缩进等等。

`color`	设置文本颜色
`direction`	设置文本方向。
`line-height`	设置行高。
`letter-spacing`	设置字符间距。
`text-align`	对齐元素中的文本。
`text-decoration`	向文本添加修饰。
`text-indent`	缩进元素中文本的首行。
`text-transform`	控制元素中的字母。
`white-space`	设置元素中空白的处理方式。
`word-spacing`	设置字间距。

**列表**

CSS 列表属性允许你放置、改变列表项标志，或者将图像作为列表项标志。

`list-style`	简写属性。用于把所有用于列表的属性设置于一个声明中。
`list-style-image`	将图象设置为列表项标志。
`list-style-position`	设置列表中列表项标志的位置。
`list-style-type`	设置列表项标志的类型

实例

    <html>
    <head>
    <style type="text/css">
    ul 
    {
    list-style: square inside url('/i/eg_arrow.gif')
    }
    </style>
    </head>
    
    <body>
    <ul>
    <li>咖啡</li>
    <li>茶</li>
    <li>可口可乐</li>
    </ul>
    </body>
    
    </html>
 
**表格**

CSS 表格属性可以帮助您极大地改善表格的外观。

`border-collapse`	设置是否把表格边框合并为单一的边框。
`border-spacing`	设置分隔单元格边框的距离。
`caption-side`	设置表格标题的位置。
`empty-cells`	设置是否显示表格中的空单元格。
`table-layout`	设置显示单元、行和列的算法。


**动画**

动画是使元素从一种样式逐渐变化为另一种样式的效果。  
您可以改变任意多的样式任意多的次数。  
请用百分比来规定变化发生的时间，或用关键词 "from" 和 "to"，等同于 0% 和 100%。  
0% 是动画的开始，100% 是动画的完成。  
为了得到最佳的浏览器支持，您应该始终定义 0% 和 100% 选择器。  
`@keyframes`规则用于创建动画。在`@keyframes`中规定某项 CSS 样式，就能创建由当前样式逐渐改为新样式的动画效果。
> Internet Explorer 10、Firefox 以及 Opera 支持 @keyframes 规则和 animation 属性。
Chrome 和 Safari 需要前缀 -webkit-。
注释：Internet Explorer 9，以及更早的版本，不支持 @keyframe 规则或 animation 属性。

实例

    div
    {
    animation-name: myfirst;
    animation-duration: 5s;
    animation-timing-function: linear;
    animation-delay: 2s;
    animation-iteration-count: infinite;
    animation-direction: alternate;
    animation-play-state: running;
    /* Firefox: */
    -moz-animation-name: myfirst;
    -moz-animation-duration: 5s;
    -moz-animation-timing-function: linear;
    -moz-animation-delay: 2s;
    -moz-animation-iteration-count: infinite;
    -moz-animation-direction: alternate;
    -moz-animation-play-state: running;
    /* Safari 和 Chrome: */
    -webkit-animation-name: myfirst;
    -webkit-animation-duration: 5s;
    -webkit-animation-timing-function: linear;
    -webkit-animation-delay: 2s;
    -webkit-animation-iteration-count: infinite;
    -webkit-animation-direction: alternate;
    -webkit-animation-play-state: running;
    /* Opera: */
    -o-animation-name: myfirst;
    -o-animation-duration: 5s;
    -o-animation-timing-function: linear;
    -o-animation-delay: 2s;
    -o-animation-iteration-count: infinite;
    -o-animation-direction: alternate;
    -o-animation-play-state: running;
    }

简写

    div
    {
    animation: myfirst 5s linear 2s infinite alternate;
    /* Firefox: */
    -moz-animation: myfirst 5s linear 2s infinite alternate;
    /* Safari 和 Chrome: */
    -webkit-animation: myfirst 5s linear 2s infinite alternate;
    /* Opera: */
    -o-animation: myfirst 5s linear 2s infinite alternate;
    }

[在线演示](http://runjs.cn/code/df8chxwp)	

**边框**

通过 CSS3，您能够创建圆角边框，向矩形添加阴影，使用图片来绘制边框。

 - border-radius
 - box-shadow
 - border-image

实例

	#div1
	{
	text-align:center;
	border:2px solid #a1a1a1;
	padding:10px 40px; 
	background:#dddddd;
	width:100px;
	border-radius:25px;
	box-shadow: 10px 10px 5px #888888;
	}
	#round
	{
	background:white;
	border:15px solid;
	-moz-border-image:url(http://www.w3school.com.cn/i/border.png) 30 30 round;	/* Old Firefox */
	-webkit-border-image:url(http://www.w3school.com.cn/i/border.png) 30 30 round;	
	/* Safari and Chrome */
	-o-border-image:url(http://www.w3school.com.cn/i/border.png) 30 30 round;		/* Opera */
	border-image:url(http://www.w3school.com.cn/i/border.png) 30 30 round;
	}
 
[在线演示](http://runjs.cn/code/xaeim085)	

**背景**

CSS 允许应用纯色作为背景，也允许使用背景图像创建相当复杂的效果。
CSS 在这方面的能力远远在 HTML 之上。

`background-attachment`	背景图像是否固定或者随着页面的其余部分滚动
`background-color`	设置元素的背景颜色
`background-image`	把图像设置为背景，CSS3 允许为元素使用多个背景图像
`background-position`	设置背景图像的起始位置
`background-repeat`	设置背景图像是否及如何重复
`background-clip`	规定背景的绘制区域 CSS3
`background-origin`	规定背景图片的定位区域 CSS3
`background-size`	规定背景图片的尺寸 CSS3

[在线演示](http://runjs.cn/code/wtfnwmzl)

**字体**

CSS 字体属性定义文本的字体系列、大小、加粗、风格（如斜体）和变形（如小型大写字母）。
在 CSS3 之前，web 设计师必须使用已在用户计算机上安装好的字体。
通过 CSS3，web 设计师可以使用他们喜欢的任意字体。

`font-family`	设置字体系列。
`font-size`	设置字体的尺寸。
`font-style`	设置字体风格。
`font-variant`	以小型大写字体或者正常字体显示文本。
`font-weight`	设置字体的粗细。

在新的 `@font-face` 规则中，您必须首先定义字体的名称（比如 myFirstFont），然后指向该字体文件。
如需为 HTML 元素使用字体，请通过 `font-family` 属性来引用字体的名称 (myFirstFont)：

    <style> 
    @font-face
    {
    font-family: myFirstFont;
    src: url('Sansation_Light.ttf'),
         url('Sansation_Light.eot'); /* IE9+ */
    }
    
    div
    {
    font-family:myFirstFont;
    }
    </style>

[http://stackoverflow.com/questions/15249654/chrome-not-respecting-font-weight](http://stackoverflow.com/questions/15249654/chrome-not-respecting-font-weight)
[在线演示](http://runjs.cn/code/dnpk3lql)
	
**多列**

通过 CSS3，您能够创建多个列来对文本进行布局 - 就像报纸那样！

 `column-count` 规定元素应该被分隔的列数
 `column-gap` 规定列之间的间隔
 `column-rule` 设置所有 column-rule-* 属性的简写属性（颜色，样式，宽度）

[在线演示](http://runjs.cn/code/5ezucw8x)

**用户界面**

在 CSS3 中，新的用户界面特性包括重设元素尺寸、盒尺寸以及轮廓等。

`resize` 规定是否可由用户对元素的尺寸进行调整。
`box-sizing` 允许您以确切的方式定义适应某个区域的具体内容。
`outline-offset` 对轮廓进行偏移，并在超出边框边缘的位置绘制轮廓。

**过渡**

    div
    {
    width:100px;
    height:100px;
    background:yellow;
    transition-property:width;
    transition-duration:1s;
    transition-timing-function:linear;
    transition-delay:2s;
    /* Firefox 4 */
    -moz-transition-property:width;
    -moz-transition-duration:1s;
    -moz-transition-timing-function:linear;
    -moz-transition-delay:2s;
    /* Safari and Chrome */
    -webkit-transition-property:width;
    -webkit-transition-duration:1s;
    -webkit-transition-timing-function:linear;
    -webkit-transition-delay:2s;
    /* Opera */
    -o-transition-property:width;
    -o-transition-duration:1s;
    -o-transition-timing-function:linear;
    -o-transition-delay:2s;
    }
    
    div:hover
    {
    width:200px;
    }
    

**文本效果**

    //添加阴影
    h1
    {
    text-shadow: 5px 5px 5px #FF0000;
    }

**2D 转换**

**3D 转换**










