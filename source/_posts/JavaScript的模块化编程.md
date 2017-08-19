---
title: JavaScript的模块化编程
date: 2016-12-26 15:52:59
tags:
---

学习前端快半年了，惊叹前端技术栈的发展之快，追逐 Angular2 ,Vue 等 MVVM 框架的同时又担忧自己的基础知识不够扎实，时常研究一些原生 JavaScript 技术。

看完 [JavaScript 模块化七日谈](https://huangxuan.me/js-module-7day/#/) 一文，更加感叹前端领域的发展之快，不求对过往的技术有多么深入的理解，了解新技术产生的背景和它们解决的问题也是非常必要的。

文中所提到的诸如 LABjs / YUI3 / Sea.js / RequireJS / Browserify 等技术我大多没有使用过，为了解 JavaScript 模块化编程的发展，记录在此。

## 从设计模式说起

### 最早，我们这么写代码

	function foo(){
	    //...
	}
	function bar(){
	    //...
	}

Global 被污染，很容易命名冲突

### 简单封装，Namespace 模式

	var MYAPP = {
	    foo: function(){},
	    bar: function(){}
	}
	
	MYAPP.foo();
	
减少 Global 上的变量数目

本质是对象，一点都不安全

### 匿名闭包：IIFE 模式

	var Module = (function(){
	    var _private = "safe now";
	    var foo = function(){
	        console.log(_private)
	    }
	
	    return {
	        foo: foo
	    }
	})()
	
	Module.foo();
	Module._private; // undefined

函数是 JavaScript 唯一的 Local Scope

### 再增强一点 ：引入依赖

	var Module = (function($){
	    var _$body = $("body");     // we can use jQuery now!
	    var foo = function(){
	        console.log(_$body);    // 特权方法
	    }
	
	    // Revelation Pattern
	    return {
	        foo: foo
	    }
	})(jQuery)
	
	Module.foo();

这就是模块模式
也是现代模块实现的基石

## 只有封装性可不够，我们还需要加载

### 回到 Script 标签

	body
	    script(src="jquery.js")
	    script(src="app.js")    // do some $ things...

DOM 顺序即执行顺序

### 但现实是这样的...

	body
	    script(src="zepto.js")
	    script(src="jhash.js")
	    script(src="fastClick.js")
	    script(src="iScroll.js")
	    script(src="underscore.js")
	    script(src="handlebar.js")
	    script(src="datacenter.js")
	    script(src="deferred.js")
	    script(src="util/wxbridge.js")
	    script(src="util/login.js")
	    script(src="util/base.js")
	    script(src="util/city.js")
	    script(src="util/date.js")
	    script(src="util/cookie.js")
	    script(src="app.js")

- 难以维护 Very difficult to maintain!  
- 依赖模糊 Unclear Dependencies  
- 请求过多 Too much HTTP calls  

### LABjs -  Script Loader（2009）

[LABjs](https://github.com/getify/LABjs)  

	$LAB.script("framework.js").wait()
	    .script("plugin.framework.js")
	    .script("myplugin.framework.js").wait()
	    .script("init.js");
 
## 模块化架构的工业革命

### YUI3 Loader - Module Loader（2009）

[Creating YUI Modules](http://yuilibrary.com/yui/docs/yui/create.html)

	// YUI - 编写模块
	YUI.add('dom', function(Y) {
	  Y.DOM = { ... }
	})
	
	// YUI - 使用模块
	YUI().use('dom', function(Y) {
	  Y.DOM.doSomeThing();
	  // use some methods DOM attach to Y
	})

Creating Custom Modules

	// hello.js
	YUI.add('hello', function(Y){
	    Y.sayHello = function(msg){
	        Y.DOM.set(el, 'innerHTML', 'Hello!');
	    }
	},'3.0.0',{
	    requires:['dom']
	})
	
	// main.js
	YUI().use('hello', function(Y){
	    Y.sayHello("hey yui loader");
	})
	
## 跳出浏览器

### CommonJS - API Standard（2009）

[CommonJS](http://www.commonjs.org/) 定义的模块分为:{模块引用(require)} {模块定义(exports)} {模块标识(module)}

require()用来引入外部模块；

exports对象用于导出当前模块的方法或变量，唯一的导出口；

module对象就代表模块本身。

	//sum.js
	exports.sum = function(){...做加操作..};
	
	//calculate.js
	var math = require('sum');
	exports.add = function(n){
	    return math.sum(val,n);
	};
	
**NodeJS : Simple HTTP Server**

	// server.js
	var http = require("http"),
	    PORT = 8000;
	
	http.createServer(function(req, res){
	    res.end("Hello World");
	}).listen(PORT);
	
	console.log("listenning to " + PORT);

**同步 / 阻塞式加载**

	// timeout.js
	var EXE_TIME = 2;
	
	(function(second){
	    var start = +new Date();
	    while(start + second*1000 > new Date()){}
	})(EXE_TIME)
	
	console.log("2000ms executed")	
	
	// main.js
	require('./timeout');   // sync load
	console.log('done!');

## 浏览器环境模块化方案（AMD/CMD）	

>[前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588)  
[前端模块化开发的价值](https://github.com/seajs/seajs/issues/547)
	
### AMD(Async Module Definition)

CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。[AMD 规范](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition) 则是非同步加载模块，允许指定回调函数。

AMD (异步模块定义)只有一个接口：define(id?,dependencies?,factory);

	define(function(){
	     var exports = {};
	     exports.method = function(){...};
	     return exports;
	 });
	 
[RequireJS (2011)](http://requirejs.org/) 对模块定义的规范化产出

### CMD(Common Module Definition)

[CMD 规范](https://github.com/cmdjs/specification/blob/master/draft/module.md)  

[SeaJS (2011)](http://seajs.org/) 对模块定义的规范化产出。

Sea.js 的用法不再介绍，借用 Sea.js 作者玉伯的 [一句评论:](https://github.com/seajs/seajs/issues/1605#issuecomment-149220246)

>任何一个技术产品，都有其生命周期，随着 ES6、ES7、webpack、babel 等技术与工具的兴起，Sea.js 也好，RequireJS 也好，都有了更好的解决方案。

## 大势所趋，去掉这层包裹


### NPM（Node Package Manger）

[NPM](https://www.npmjs.com/) 是 Node.js 的模块依赖管理工具，我们可以使用 NPM 来发布或下载代码。模块化是是 NPM 背后的核心原则。当需要某个特定的功能时，我们能安装相应的模块并加载到应用当中。

### Browserify - CommonJS In Browser (2011 / 2014 stable)

前面提到的 Sea.js / RequireJS 是一种在线"编译" 模块的方案，相当于在页面上加载一个 CMD / AMD 解释器。这样浏览器就认识了 define、exports、module 这些东西，也就实现了模块化。
而 Browserify / Webpack 是一个预编译模块的方案，相比于上面 ，这个方案更加智能，不需要在浏览器中加载解释器。

### Webpack - Module Bundler (2014)

<img src="https://webpack.github.io/assets/what-is-webpack.png" width = "380" alt="Webpack"/>

[Webpack](https://webpack.js.org/) 是一个用于现代 JavaScript 应用的模块打包器(module bundler)，它能够分析你的项目结构，找到 JavaScript 模块以及一些浏览器无法运行的语言（Scss，TypeScript等），并将其打包成合适的格式供浏览器使用。 

*以下实例来自 [vue-cli](https://github.com/vuejs/vue-cli)*

### 使用 Loaders

	loaders: [
	      {
	        test: /\.vue$/,
	        loader: 'vue'
	      },
	      {
	        test: /\.js$/,
	        loader: 'babel',
	        include: projectRoot,
	        exclude: /node_modules/
	      },
	      {
	        test: /\.json$/,
	        loader: 'json'
	      }
	    ]

### 使用插件

	plugins: [
	    new webpack.DefinePlugin({
	      'process.env': config.dev.env
	    }),
	    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
	    new webpack.optimize.OccurrenceOrderPlugin(),
	    new webpack.HotModuleReplacementPlugin(),
	    new webpack.NoErrorsPlugin(),
	    // https://github.com/ampedandwired/html-webpack-plugin
	    new HtmlWebpackPlugin({
	      filename: 'index.html',
	      template: 'index.html',
	      inject: true
	    })
	  ]	

[代码示例](https://github.com/bigggge/study-webpack)	
## ES6 模块

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代现有的 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

	// lib/math.js
	export function sum(x, y) {
	  return x + y;
	}
	export var pi = 3.141593;
	
	// app.js
	import * as math from "lib/math";
	console.log("2π = " + math.sum(math.pi, math.pi));
	
	// otherApp.js
	import {sum, pi} from "lib/math";
	console.log("2π = " + sum(pi, pi));
	
	// lib/mathplusplus.js
	export * from "lib/math";
	export var e = 2.71828182846;
	export default function(x) {
	    return Math.exp(x);
	}
	
	// app.js
	import exp, {pi, e} from "lib/mathplusplus";
	console.log("e^π = " + exp(pi));

更多 ES6 特性参考 [ECMAScript 6 入门](http://es6.ruanyifeng.com/) 和 [代码示例](https://github.com/bigggge/JsPractice/tree/master/src/es6) 
### Babel - JavaScript Compiler (2015)

[Babel](http://babeljs.io/) 是一个 JavaScript 编译器。默认情况下，Babel 自带了一组 ES2015 语法转化器。这些转化器能让你现在就使用最新的 JavaScript 语法，而不用等待浏览器提供支持。

[代码示例](https://github.com/bigggge/study-webpack/tree/master/babel)

### TypeScript - JavaScript 的超集 (2012 / 2016 TypeScript 2.0)

[TypeScript](https://www.typescriptlang.org/) 提供最新的 JavaScript 特性，包括那些来自 ECMAScript2015 和未来的提案中的特性，以帮助建立健壮的组件。

这些特性为高可信应用程序开发时是可用的，但是会被 TypeScript 编译器编译成简洁的ECMAScript3（或更新版本）的 JavaScript。TypeScript 在 [Angular2](https://angular.io/) 开发中非常流行。 

参考资料：

[JavaScript 模块化七日谈](https://huangxuan.me/js-module-7day/#/)  
[浅析JS中的模块规范（CommonJS，AMD，CMD）](http://www.cnblogs.com/skylar/p/4065455.html)  
[Webpack、Browserify和Gulp三者之间到底是怎样的关系？](https://www.zhihu.com/question/37020798)  
[入门Webpack，看这篇就够了](http://www.jianshu.com/p/42e11515c10f#)    
[ECMAScript 6 入门](http://es6.ruanyifeng.com/) 







