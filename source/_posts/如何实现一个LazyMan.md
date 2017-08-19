---
title: 如何实现一个LazyMan?
date: 2017-01-19 13:17:38
tags:
---
> 实现一个LazyMan，可以按照以下方式调用:
LazyMan(“Hank”)输出:  
Hi! This is Hank!  
LazyMan(“Hank”).sleep(10).eat(“dinner”)输出  
Hi! This is Hank!  
//等待10秒..  
Wake up after 10  
Eat dinner~  
LazyMan(“Hank”).eat(“dinner”).eat(“supper”)输出  
Hi This is Hank!  
Eat dinner~  
Eat supper~  
LazyMan(“Hank”).sleepFirst(5).eat(“supper”)输出  
//等待5秒  
Wake up after 5  
Hi This is Hank!  
Eat supper  
以此类推。  

这是一道来自微信的 JavaScript 面试题，对于这个问题，我们首先想到使用一个队列来维护任务列表，通过返回 this 来实现链式调用，于是便有了下面的代码：

	function LazyMan(name) {
	
	    this.tasks = []
	    var _this = this
	
	    var fn = (function () {
	        console.log(`Hi! This is ${name}!`)
	    })
	
	    this.tasks.push(fn)
	    _this._next()
	}
	
	LazyMan.prototype._next = function () {
	    var fn = this.tasks.shift()
	    fn && fn()
	}
	
	LazyMan.prototype.eat = function (name) {
	    var _this = this
	
	    var fn = (function () {
	        console.log(`Eat ${name}`)
	    })
	    this.tasks.push(fn)
	    _this._next()
	
	    return this
	}
	
	LazyMan.prototype.sleep = function (time) {
	    var _this = this
	
	    var fn = (function () {
	        setTimeout(function () {
	            console.log(`Wake up after ${time} s`)
	        }, time * 1000)
	    })
	
	    this.tasks.push(fn)
	    _this._next()
	
	    return this
	}
	
	new LazyMan('GGG').eat('fish').sleep(1)
	
	
我们来测一测结果

![](http://7xq3d5.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-01-19%20%E4%B8%8A%E5%8D%8811.54.01.png)

好像没有问题，再试试 `new LazyMan('GGG').sleep(1).eat('fish')` 发现还是上图的结果。

怎么还是同一个结果呢？检查代码，发现原来是`_next()`方法放错了位置，我们应该将其放入`fn`函数内才对。值得注意的一点是，setTimeout 函数会在一个时间段过去后在队列中添加一个消息。（详见：[浏览器的工作原理](https://blog.haoduoyu.cc/2016/12/06/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/)）所以为了在任务队列中先 push 完所有任务然后再依次执行，我们需要添加上 setTimeout 函数。全部代码如下：

	function _LazyMan(name) {
	
	    this.tasks = []
	    var _this = this
	
	    var fn = (function () {
	        console.log(`Hi! This is ${name}!`)
	        _this._next()
	    })
	
	    this.tasks.push(fn)
	
	    setTimeout(function () {
	        _this._next()
	    }, 0)
	}
	
	_LazyMan.prototype._next = function () {
	    var fn = this.tasks.shift()
	    fn && fn()
	}
	
	_LazyMan.prototype.eat = function (name) {
	    var _this = this
	
	    var fn = (function () {
	        console.log(`Eat ${name}`)
	        _this._next()
	    })
	
	    this.tasks.push(fn)
	    return this
	}
	
	_LazyMan.prototype.sleep = function (time) {
	    var _this = this
	
	    var fn = (function () {
	        setTimeout(function () {
	            console.log(`Wake up after ${time} s`)
	            _this._next()
	        }, time * 1000)
	    })
	
	    this.tasks.push(fn)
	    return this
	}
	
	_LazyMan.prototype.sleepFirst = function (time) {
	    var _this = this
	
	    var fn = (function () {
	        setTimeout(function () {
	            console.log(`Wake up after ${time} s`)
	            _this._next()
	        }, time * 1000)
	    })
	
	    this.tasks.unshift(fn)
	    return this
	}
	
	function LazyMan(name) {
	    return new _LazyMan(name)
	}
	
	LazyMan('GGG').sleep(3).eat('milk').eat('fish').sleep(2).eat('water').sleepFirst(2)


代码来自：[如何实现一个LazyMan?](http://natumsol.github.io/2016/09/09/lazyman/)