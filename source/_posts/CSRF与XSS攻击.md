---
title: CSRF 与 XSS 攻击
date: 2017-02-08 14:15:05
tags:
---

## 发起攻击

假如某直播网站赠送礼物的接口是：

    https://xxx.com/gift/send?target=someone&giftId=123

用户只要在登录状态下请求这个地址就能给 someone 赠送礼物 123，攻击者可以通过恶意链接或者将链接填充到`img`标签中就能发起攻击。

如果是一个 POST 请求：

    https://xxx.com/gift/deleteRecord

请求参数为`{giftId : 123}`,攻击者则可以通过构造恶意页面发起攻击。

	<body>
	<iframe name="hiddenIframe" style="display:none"></iframe>
	<!--target 属性规定在何处打开 action URL-->
	<form action="https://haoduoyu.cc/gift/deleteRecord" id="form" method="post" style="visibility:hidden"
	      target="hiddenIframe">
	    <input type="text" name="giftId" value="123">
	</form>
	<script>
	    document.getElementById('form').submit();
	    location.href = "https://haoduoyu.cc";
	</script>
	</body>

当用户点开这个页面的链接后，会自动发送 POST 请求，然后跳转到原始页面。

## CSRF 攻击

上述攻击就是典型的 CSRF 攻击。CSRF（Cross Site Request Forgery, 跨站域请求伪造）是一种网络的攻击方式，其原理是攻击者构造网站后台某个功能接口的请求地址，诱导用户去点击或者用特殊方法让该请求地址自动加载。

## 防御 CSRF 

### 验证 HTTP Referer 字段

Referer 字段记录了该 HTTP 请求的来源地址，我们可以通过验证来源网站的地址来确定请求的合法性。但缺点是某些浏览器可以篡改 Referer 的值，且服务器转发也会改变 Referer。另一方面，Referer 值会记录下用户的访问来源，有些用户认为这样会侵犯到他们自己的隐私权。

### 在请求地址中添加 token 并验证

CSRF 攻击之所以能够成功，是因为攻击者可以完全伪造用户的请求，该请求中所有的用户验证信息都是存在于 cookie 中，因此攻击者可以在不知道这些验证信息的情况下直接利用用户自己的 cookie 来通过安全验证。要抵御 CSRF，关键在于在请求中放入黑客所不能伪造的信息，并且该信息不存在于 cookie 之中。可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。

缺点是难以保证 token 本身的安全。攻击者可以在论坛之类的网站发布网站的地址。由于系统也会在这个地址后面加上 token，攻击者可以在自己的网站上得到这个 token，并马上就可以发动 CSRF 攻击。为了避免这一点，系统可以在添加 token 的时候增加一个判断，如果这个链接是链到自己本站的，就在后面添加 token，如果是通向外网则不加。不过，即使这个 csrftoken 不以参数的形式附加在请求之中，黑客的网站也同样可以通过 Referer 来得到这个 token 值以发动 CSRF 攻击。

### 在 HTTP 头中自定义属性并验证

这种方法也是使用 token 并进行验证，和上一种方法不同的是，这里并不是把 token 以参数的形式置于 HTTP 请求之中，而是把它放到 HTTP 头中自定义的属性里。

## XSS 攻击

跨站脚本（Cross-site scripting，XSS）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。这类攻击通常包含了HTML以及用户端脚本语言。

Web 应用未对用户提交请求的数据做充分的检查过滤，允许用户在提交的数据中掺入 HTML 代码(最主要的是`>`、`<`)，并将未经转义的恶意代码输出到第三方用户的浏览器解释执行，是导致XSS漏洞的产生原因。

防御 XSS 攻击最主要的方法是过滤特殊字符，进行 HTML 转义。

### 存储型 XSS

存储型XSS，持久化，代码是存储在服务器中的，如在个人信息或发表文章等地方，加入代码，如果没有过滤或过滤不严，那么这些代码将储存到服务器中，用户访问该页面的时候触发代码执行。这种XSS比较危险，容易造成蠕虫，盗窃cookie等。

### 反射型 XSS

反射型XSS，非持久化，需要欺骗用户自己去点击链接才能触发XSS代码（服务器中没有这样的页面和内容），一般容易出现在搜索页面。

## DVWA 

[DVWA](https://github.com/ethicalhack3r/DVWA) 是一款基于 PHP/MySQL 编写的用于常规 WEB 漏洞教学和脆弱性测试的应用。 可以帮助我们方便的检测 CSRF/XSS 漏洞。

![dvwa](http://7xq3d5.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-08%20%E4%B8%8B%E5%8D%883.28.02.png?imageView2/2/w/800)

参考资料：

[「每日一题」CSRF 是什么？](https://zhuanlan.zhihu.com/p/22521378)    
[CSRF 攻击的应对之道](https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/)  
[避免XSS攻击](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/09.3.md)