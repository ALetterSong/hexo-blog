---
title: CORS 跨域资源共享
date: 2016-11-20 17:05:32
tags:
---
### 概述 

通过 XHR 实现 AJAX 通信的一个主要限制，来源于跨域安全策略。默认情况下，XHR对象只能访问与包含它的页面位于同一个域中的资源( [浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy):如果协议，端口和主机都相同)。这种安全策略可以预防某些恶意行为。但是，实现合理的跨域请求对开发某些浏览器应用程序也是至关重要的。

CORS（Cross-Origin Resource Sharing，跨域资源共享）是 W3C 的一个工作草案，定义了在必须访问跨源资源时，浏览器与服务器应该如何沟通。CORS 背后的的基本思想，就是使用自定义的 HTTP 头部让浏览器与服务器进行沟通，从而决定请求或响应的成功和失败。

CORS 需要浏览器和服务器同时支持。目前，大部分浏览器都支持该功能 [(浏览器支持情况)](http://caniuse.com/#search=cors)。CORS 通信与同源的 AJAX 通信没有差别。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。

### 访问控制场景

#### 简单请求

所谓的简单，是指：

- 只使用 GET, HEAD 或者 POST 请求方法。如果使用 POST 向服务器端传送数据，则数据类型(Content-Type)只能是 `application/x-www-form-urlencoded`, `multipart/form-data` 或 `text/plain` 中的一种
- 不会使用自定义请求头（类似于 X-Modified 这种）

下面是一个使用简单请求的例子:

	   $.ajax({
	      url: 'http://localhost:8002/listUsers',
	      method: 'GET',
	      success: function (data) {
	          p.innerHTML = JSON.stringify(data);
	          document.body.appendChild(p);
	      }
	   });
	   
	   
	   
Request Headers:

	Accept:*/*
	Accept-Encoding:gzip, deflate, sdch, br
	Accept-Language:zh-CN,zh;q=0.8,en;q=0.6
	Cache-Control:no-cache
	Connection:keep-alive
	Host:localhost:8002
	Origin:http://localhost:63343
	Pragma:no-cache
	Referer:http://localhost:63343/JsPractice/src/js/ajax/json.html?_ijt=4f3ur0duo147k993set7na2733
	User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36	   

上面的头信息中，Origin 字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。
如果 Origin 指定的源，不在许可范围内，服务器会返回一个正常的 HTTP 回应。浏览器发现，这个回应的头信息没有包含 Access-Control-Allow-Origin 字段，就知道出错了，从而抛出一个错误，被 XMLHttpRequest 的 onerror 回调函数捕获。

这种错误无法通过状态码识别，因为 HTTP 回应的状态码有可能是200。

如果Origin指定的域名在许可范围内，服务器返回的响应，会多出几个头信息字段。请看 Response Headers:

    Access-Control-Allow-Origin:*
    Connection:keep-alive
    Content-Length:1138
    Content-Type:application/json; charset=utf-8
    Date:Sun, 20 Nov 2016 07:57:09 GMT
    X-Powered-By:Express

Access-Control-Allow-Origin 字段是必须的。它的值要么是请求时 Origin 字段的值，要么是一个 *，表示接受任意域名的请求。

完整的请求头和响应头参见[HTTP访问控制(CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS#HTTP响应头)	
	
#### 非简单请求(预请求)

- 请求以 GET, HEAD 或者 POST 以外的方法发起请求。或者，使用 POST，但请求数据为 `application/x-www-form-urlencoded`, `multipart/form-data` 或者 `text/plain` 以外的数据类型。比如说，用 POST 发送数据类型为 `application/xml` 或者 `text/xml` 的 XML 数据的请求。
- 使用自定义请求头（比如添加诸如 X-PINGOTHER）

不同于简单请求，“预请求”要求必须先发送一个 OPTIONS 请求给目的站点，来查明这个跨站请求对于目的站点是不是安全可接受的。

在服务器端我们可以进行如下设置允许跨域访问，但是我们应该根据合适的数据（用户代理，来源页面等）来决定是否设置 Access-Control-Allow-Origin 头部，以保证安全：

	//设置跨域访问
	app.all('*', function (req, res, next) {
	    res.header("Access-Control-Allow-Origin", "*");
	    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
	    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
	    res.header("Content-Type", "application/json;charset=utf-8");
	    next();
	});

#### 

### 其他跨域技术

#### JSONP

JSONP 是 JSON with padding(填充式 JSON 或参数式 JSON )的简写，是应用 JSON 的一种新方法。JSONP 的原理非常简单，为了克服跨域问题，利用没有跨域限制的 script 标签加载预设的 callback 将内容传递给 JavaScript。

下面是一个典型的 JSONP 实例


    function handleResponse(response){
	      alert("You're at IP address " + response.ip + ", which is in " + response.city + ", " + response.region_name);
	 }
	    
	 var script = document.createElement("script");
	 script.src = "http://freegeoip.net/json/?callback=handleResponse";
	 document.body.insertBefore(script, document.body.firstChild);

jQuery实现JSONP:

	<script>
	    function doSomething(jsonData) {
	        alert('doSomething ' + jsonData.status);
	    }
	    $.ajax({
	        type: "get",
	        url: 'http://localhost:8002/jsonp',
	        dataType: "jsonp",
	        jsonp: "callback",//传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
	        jsonpCallback: "doSomething",//自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
	        success: function (json) {
	            alert(json.status);
	        },
	        error: function () {
	            alert('fail');
	        }
	    });
	</script>
	
服务端：

	app.get('/jsonp', function (req, res) {
	    res.header("Access-Control-Allow-Origin", "*");
	    res.jsonp({status: 'jsonpStatus'});
	});	

#### window.name

window 对象有个 name 属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个 window.name 的，每个页面对 window.name 都有读写的权限，window.name 是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。

a.html

	<script>
	    window.name = "我是页面a设置的值";
	    setTimeout(function () {
	        window.location = "b.html";
	    })
	</script>

b.html

	<script>
	    alert(window.name);
	</script>


#### window.postMessage

window.postMessage(message,targetOrigin)  方法是html5新引进的特性，可以使用它来向其它的window对象发送消息，无论这个window对象是属于同源或不同源，目前IE8+、FireFox、Chrome、Opera等浏览器都已经支持window.postMessage方法。

a.html

	<script>
	    function onLoad() {
	        var iframe = document.getElementById('iframe');
	        var win = iframe.contentWindow;
	        win.postMessage("我来自页面a", "*");
	    }
	</script>
	<iframe id="iframe" src="b.html" onload="onLoad()"></iframe>

b.html

	<script>
	    window.onmessage = function (e) {
	        e = e || event;
	        alert(e.data);
	    }
	</script>


参考资料：

[JavaScript 高级程序设计 ch21]()  
[HTTP访问控制(CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)  
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)  
[Node.js express 跨域问题](https://cnodejs.org/topic/51dccb43d44cbfa3042752c8)  
[js中几种实用的跨域方法原理详解](http://www.cnblogs.com/2050/p/3191744.html)  
[说说JSON和JSONP，也许你会豁然开朗，含jQuery用例](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)


