title: hello,前端－HTML篇
date: 2016-08-15 23:12:04
tags:
---



> **hello,前端－HTML篇**  
hello,前端－CSS篇   
...  
本文大部分内容总结自[w3school](http://www.w3school.com.cn/css/index.asp)。   
本系列是我在学习前端过程中的一系列总结和整理。

# HTML基础

## 什么是HTML

 - HTML 指的是超文本标记语言 (Hyper Text Markup Language)
 - HTML 不是一种编程语言，而是一种标记语言 (markup language)
 - 标记语言是一套标记标签 (markup tag)
 - HTML 使用标记标签来描述网页

## 基本结构
网页是由一系列的标签组成的。    
例如

    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>标题</title>
    </head>
    <body>
      <h1>My First Heading</h1>
      <p>My first paragraph.</p>
    </body>
    </html>

## HTML元素（标签）
HTML 文档是由 HTML 元素定义的。
    
HTML 元素指的是从开始标签（start tag）到结束标签（end tag）的所有代码。

HTML 元素语法

 - HTML 元素以开始标签起始
 - HTML 元素以结束标签终止
 - 元素的内容是开始标签与结束标签之间的内容
 - 某些 HTML 元素具有空内容（empty content）
 - 空元素在开始标签中进行关闭（以开始标签的结束而结束），如 `<br/>`
 - 大多数 HTML 元素可拥有属性

## 块级元素和内联元素
大多数 HTML 元素被定义为块级元素或内联元素。
块级元素在浏览器显示时，通常会以新行来开始（和结束）。
如：`<div>`,`<h1>`, `<p>`, `<ul>`, `<table>`    
内联元素在显示时通常不会以新行开始。
如：`<span>`,`<b>`, `<td>`, `<a>`, `<img>`

## HTML语义化
语义化的含义就是用正确的标签做正确的事情，html语义化就是让页面的内容结构化，便于对浏览器、搜索引擎解析；在没有样式CCS情况下也以一种文档格式显示，并且是容易阅读的。搜索引擎的爬虫依赖于标记来确定上下文和各个关键字的权重，利于 SEO。使阅读源代码的人对网站更容易将网站分块，便于阅读维护理解。


# HTML标签
 `<!--...-->` 定义注释
 `<!DOCTYPE html> ` 定义文档类型（html5）
 `<a>` 定义超链接
*href 规定链接指向的页面的 URL*
 `<body>` 定义文档的主体，包含文档的所有内容
 `<br>` 定义简单的折行，是一个空标签
 `<div>` 定义文档中的节，是一个块极元素，它的内容自动开始一个新行
 `<dl>` 定义定义列表，用于结合 `<dt>` （定义列表中的项目）和 `<dd>` （描述列表中的项目）
 `<dt>` 定义定义列表中的项目
 `<dd>` 定义定义列表中项目的描述

    <dl>
       <dt>计算机</dt>
       <dd>用来计算的仪器 ... ...</dd>
       <dt>显示器</dt>
       <dd>以视觉方式显示信息的装置 ... ...</dd>
    </dl>
 `<em>` 定义强调文本
 `<form>` 定义供用户输入的 HTML 表单
 
    <form action="form_action.asp" method="get">
      <p>First name: <input type="text" name="fname" /></p>
      <p>Last name: <input type="text" name="lname" /></p>
      <input type="submit" value="Submit" />
    </form>
 `<frame>` 定义框架集的窗口或框架
 `<h1> to <h6>` 定义 HTML 标题
 `<head>` 定义关于文档的信息，`<head>` 中的元素可以引用脚本、指示浏览器在哪里找到样式表、提供元信息等等
 `<html>` 定义 HTML 文档
 `<img>` 定义图像
*alt 规定图像的替代文本*
*src 规定显示图像的 URL*
 `<input>` 定义输入控件
*type 规定input元素的[类型](http://www.w3school.com.cn/tags/att_input_type.asp)*
 `<label>` 定义 input 元素的标注，如果在 label 元素内点击文本，就会触发此控件
*for 规定 label 绑定到哪个表单元素*
 `<li>` 定义列表的项目
 `<ol>` 定义有序列表
 `<ul>` 定义无序列表
   
        <ol>
           <li>Coffee</li>
           <li>Tea</li>
           <li>Milk</li>
        </ol>
        <ul>
           <li>Coffee</li>
           <li>Tea</li>
           <li>Milk</li>
        </ul>

 `<link>` 定义文档与外部资源的关系,它是空元素，仅包含属性，最常见的用途是链接样式表
 `<meta>` 定义关于 HTML 文档的元信息

    <meta charset='utf-8'> 
    <!-- 声明文档使用的字符编码 -->

 `<p>` 定义段落
 `<script>` 定义客户端脚本，如JavaScript
 `<span>` 定义文档中的节，被用来组合文档中的行内元素
 `<strong>` 定义强调文本
 `<style>` 定义文档的样式信息
 `<table>` 定义表格
 `<tr>` 定义表格中的行
 `<th>` 定义表格中的表头单元格
 `<td>` 定义表格中的单元
 
    <table border="1">
      <tr>
        <th>Month</th>
        <th>Savings</th>
      </tr>
      <tr>
        <td>January</td>
        <td>$100</td>
      </tr>
    </table>
 `<textarea>` 定义多行的文本输入控件
 `<title>` 定义文档的标题

# DOM

HTML DOM 定义了访问和操作 HTML 文档的标准方法。
DOM 将 HTML 文档表达为树结构。

HTML DOM 树
![DOM树](http://www.w3school.com.cn/i/ct_htmltree.gif)

## 什么是DOM

DOM 是 W3C（万维网联盟）的标准。
DOM 定义了访问 HTML 和 XML 文档的标准：

> “W3C 文档对象模型 （DOM） 是中立于平台和语言的接口，它允许程序和脚本动态地访问和更新文档的内容、结构和样式。”

W3C DOM 标准被分为 3 个不同的部分：

 - 核心 DOM - 针对任何结构化文档的标准模型
 - XML DOM - 针对 XML 文档的标准模型
 - HTML DOM - 针对 HTML 文档的标准模型

DOM 是 *Document Object Model*（文档对象模型）的缩写。

## 什么是HTML DOM

HTML DOM 是：

 - HTML 的标准对象模型
 - HTML 的标准编程接口
 - W3C 标准

HTML DOM 定义了所有 HTML 元素的对象和属性，以及访问它们的方法。
换言之，HTML DOM 是关于如何获取、修改、添加或删除 HTML 元素的标准。

## 节点的关系

节点树中的节点彼此拥有层级关系。
父（parent）、子（child）和同胞（sibling）等术语用于描述这些关系。父节点拥有子节点。同级的子节点被称为同胞（兄弟或姐妹）。

 - 在节点树中，顶端节点被称为根（root）

 - 每个节点都有父节点、除了根（它没有父节点）
 - 一个节点可拥有任意数量的子
 - 同胞是拥有相同父节点的节点

下面的图片展示了节点树的一部分，以及节点之间的关系：
![节点的关系](http://www.w3school.com.cn/i/dom_navigate.gif)

## DOM方法和属性
### 方法
`getElementById()`  返回带有指定 ID 的元素。
`getElementsByTagName()`  返回包含带有指定标签名称的所有元素的节点列表（集合/节点数组）。
`getElementsByClassName()`  返回包含带有指定类名的所有元素的节点列表。
`appendChild()` 把新的子节点添加到指定节点。
`removeChild()` 删除子节点。
`replaceChild()`  替换子节点。
`insertBefore()`  在指定的子节点前面插入新的子节点。
`createAttribute()`创建属性节点。
`createElement()` 创建元素节点。
`createTextNode()`  创建文本节点。
`getAttribute()`  返回指定的属性值。
`setAttribute()`  把指定属性设置或修改为指定的值。
### 属性
`innerHTML`- 节点（元素）的文本值
`parentNode` - 节点（元素）的父节点
`childNodes` - 节点（元素）的子节点
`attributes` - 节点（元素）的属性节点
`nodeName` - 节点（元素）的名称
*元素节点的 nodeName 与标签名相同*
*属性节点的 nodeName 与属性名相同*
*文本节点的 nodeName 始终是 #text*
*文档节点的 nodeName 始终是 #document*
`nodeValue` - 节点（元素）的值
*元素节点的 nodeValue 是 undefined 或 null*
*文本节点的 nodeValue 是文本本身*
*属性节点的 nodeValue 是属性值*
`nodeType` - 节点（元素）的属性节点
*元素 1*，*属性  2*，*文本  3*，*注释  8*，*文档  9*


[HTML标签列表](http://www.w3school.com.cn/tags/index.asp)
[HTML DOM对象参考手册](http://www.w3school.com.cn/jsref/dom_obj_document.asp)
[常用的 HTML头部标签](https://github.com/yisibl/blog/issues/1)
[DOCTYPE和浏览器渲染模式](https://leohxj.gitbooks.io/front-end-database/content/html-and-css-basic/doctype-and-brower-render.html)
[HTML语义化](https://leohxj.gitbooks.io/front-end-database/content/html-and-css-basic/semantic-html.html)


