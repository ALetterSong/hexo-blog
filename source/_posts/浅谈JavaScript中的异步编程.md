---
title: 浅谈JavaScript中的异步编程
date: 2016-12-05 17:00:48
tags:
---


我们知道，JavaScript 的执行环境是单线程的。所谓单线程，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务，以此类推。

这种模式的好处是实现起来比较简单；坏处是只要有一个任务耗时很长，后面的任务都必须排队等着，会拖延整个程序的执行。代码长时间运行甚至能导致浏览器无响应。

为了解决这个问题，JavaScript 语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）。

### 同步和异步

**同步**：如果在函数A返回的时候，调用者就能够得到预期结果(即拿到了预期的返回值或者看到了预期的效果)，那么这个函数就是同步的。

**异步**：如果在函数A返回的时候，调用者还不能够得到预期结果，而是需要在将来通过一定的手段得到，那么这个函数就是异步的。

以AJAX请求为例

**异步AJAX**：

- 主线程：“你好，AJAX线程。请你帮我发个HTTP请求吧，我把请求地址和参数都给你了。”
- AJAX线程：“好的，主线程。我马上去发，但可能要花点儿时间呢，你可以先去忙别的。”
- 主线程：：“谢谢，你拿到响应后告诉我一声啊。”
- (接着，主线程做其他事情去了。过了一会儿，它收到了响应到达的通知。)

**同步AJAX**：

- 主线程：“你好，AJAX线程。请你帮我发个HTTP请求吧，我把请求地址和参数都给你了。”
- AJAX线程：“......”
- 主线程：：“喂，AJAX线程，你怎么不说话？”
- AJAX线程：“......”
- 主线程：：“喂！喂喂喂！”
- AJAX线程：“......”
- (好久以后)
- 主线程：：“喂！求你说句话吧！”
- AJAX线程：“主线程，不好意思，我在工作的时候不能说话。你的请求已经发完了，拿到响应数据了，给你。”


### 回调函数

	var fn = function (callback) {
	    callback('callback');
	};
	
	fn(function (value) {
	    console.log(value);
	});

回调函数使用简单，容易理解，缺点就是存在耦合也容易造成 callback hell。

### 事件监听

事件监听模式中，任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

	$('#button').click(function () {
	     console.log('click');
	});
	
从本质上说，事件监听也是一种观察者模式。

### 发布/订阅

观察者模式又叫发布订阅模式（Publish/Subscribe），它定义了一种一对多的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使得它们能够自动更新自己。

	var events = (function () {
	    var topics = {};
	
	    return {
	        publish: function (topic, info) {
	            console.log('publish a topic:' + topic);
	            if (topics.hasOwnProperty(topic)) {
	                topics[topic].forEach(function (handler) {
	                    handler(info ? info : {});
	                })
	            }
	        },
	        subscribe: function (topic, handler) {
	            console.log('subscribe an topic:' + topic);
	            if (!topics.hasOwnProperty(topic)) {
	                topics[topic] = [];
	            }
	
	            topics[topic].push(handler);
	        },
	        remove: function (topic, handler) {
	            if (!topics.hasOwnProperty(topic)) {
	                return;
	            }
	
	            var handlerIndex = -1;
	            topics[topic].forEach(function (element, index) {
	                if (element === handler) {
	                    handlerIndex = index;
	                }
	            });
	
	            if (handlerIndex >= 0) {
	                ////从第 handlerIndex 位开始删除 1 个元素
	                topics[topic].splice(handlerIndex, 1);
	            }
	        },
	        removeAll: function (topic) {
	            console.log('remove all the handler on the topic:' + topic);
	            if (topics.hasOwnProperty(topic)) {
	                topics[topic].length = 0;
	            }
	        }
	    }
	})();
	
	
	//主题监听函数
	var handler = function (info) {
	    console.log(info);
	};
	
	//订阅hello主题
	events.subscribe('hello', handler);
	
	//发布hello主题
	events.publish('hello', 'hello world');

总的来说，观察者模式所做的工作就是在解耦，让耦合的双方都依赖于抽象，而不是依赖于具体。从而使得各自的变化都不会影响到另一边的变化。

### Promises对象

![promise](http://liubin.org/promises-book/Ch1_WhatsPromises/img/promise-states.png)

所谓 Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。

Promise 对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称Fulfilled）和 Rejected（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

	new Promise((resolve, reject) => {
	    setTimeout(resolve('done'), 100);
	}).then(value =>
	    console.log(value)
	).catch(error =>
	    console.log(error)
	);

参考资料：

[Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)   
[javascript 异步编程](http://www.cnblogs.com/rubylouvre/archive/2011/03/14/1982699.html)  
[探索Javascript异步编程](https://my.oschina.net/taogang/blog/267707)    
[JavaScript 异步编程学习笔记](https://github.com/riskers/blog/issues/22)  
[JavaScript：彻底理解同步、异步和事件循环(Event Loop)](https://segmentfault.com/a/1190000004322358)  
[JavaScript异步编程原理](http://www.cnblogs.com/hustskyking/p/javascript-asynchronous-programming.html) 
[JavaScript Promise迷你书（中文版）](http://liubin.org/promises-book/)   
[Javascript中常见的异步编程模型](http://blog.gejiawen.com/2015/10/12/some-javascript-async-pattern/)  

