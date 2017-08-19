title: 理解HTTP协议
date: 2016-01-20 14:09:30
tags: 网络
---

> 修改自 2016-01-20 发布的 HTTP 协议简述

### 协议概述
超文本传输协议（英文：*HyperText Transfer Protocol*，缩写：HTTP）是互联网上应用最为广泛的一种网络协议。HTTP是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型。HTTP是一个无状态的协议。

### TCP/IP

**传输控制协议**（英语：*Transmission Control Protocol*, TCP）是一种面向连接的、可靠的、基于字节流的传输层通信协议。
**用户数据报协议**（英语：*User Datagram Protocol*，缩写为UDP），是一个简单的面向数据报的传输层协议。面向无连接的，不可靠的数据报服务。
#### 三次握手
![三次握手][1]
SYN即握手信号，ACK即确认字符。
#### 分层
|  TCP/IP分层      | 协议         | 
| -------------     |-----------| 
| 应用层        | 该层包括所有和应用程序协同工作，利用基础网络交换应用程序专用的数据的协议。例如HTTP、FTP、DNS | 
| 传输层      | 传输层的协议，能够解决诸如端到端可靠性和保证数据按照正确的顺序到达这样的问题。例如TCP、UDP、RTP、SCTP      |  
| 网络互连层 | 解决在一个单一网络上传输数据包的问题。对于TCP/IP来说这是因特网协议（IP）      |    
| 网络接口层 | 是数据包从一个设备的网络层传输到另外一个设备的网络层的方法。例如以太网、Wi-Fi、MPLS等。      |   

### 请求信息（Request Message）

 - 请求行
 - （请求）头
 - 空行
 - 其他消息体

### 请求方法

[RFC 2616](https://tools.ietf.org/html/rfc2616) 定义了HTTP协议中现今广泛使用的一个版本 - HTTP 1.1
HTTP/2 标准于2015年5月以 [RFC 7540](https://tools.ietf.org/html/rfc7540) 正式发表，替换 HTTP 1.1 成为 HTTP 的实现标准。

>征求意见稿（Request For Comments，RFC），是由互联网工程任务组（IETF）发布的一系列备忘录。文件收集了有关互联网相关信息，以及 UNIX 和互联网社区的软件文件，以编号排定。目前 RFC 文件是由互联网协会（ISOC）赞助发行。

 - OPTIONS：这个方法可使服务器传回该资源所支持的所有 HTTP 请求方法。用'*'来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作。
 - HEAD：与 GET 方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。
 - GET：向指定的资源发出“显示”请求。使用 GET 方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在 Web Application 中。其中一个原因是 GET 可能会被网络蜘蛛等随意访问。
 - POST：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。
 - PUT：向指定资源位置上传其最新内容。
 - DELETE：请求服务器删除 Request-URI 所标识的资源。
 - TRACE：回显服务器收到的请求，主要用于测试或诊断。
 - CONNECT：HTTP/1.1协议中预留给能够将连接改为渠道方式的代理服务器。通常用于 SSL 加密服务器的链接（经由非加密的 HTTP 代理服务器）。

**哪些叫幂等或/且安全的方法？** 

- 安全方法是指不修改资源状态的 HTTP 方法。
- HTTP 幂等方法是指无论调用多少次都不会有不同结果的 HTTP 方法。它无论是调用一次，还是十次都无关紧要。结果仍应相同。


|HTTP Method    |Idempotent (幂等的)	|Safe(安全的)|
|-------------  |-------------       |-----------| 
|OPTIONS	      |yes                 |yes
|GET	         |yes	 					|yes
|HEAD	         |yes						|yes
|PUT	         |yes						|no
|POST	         |no						|no
|DELETE	      |yes					|no
|PATCH	         |no						|no

### RESTful架构

**RESTful架构**(Representational State Transfer)的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"资源"（Resources）的"表现层"。

- 所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。比如一段文本，一张图片。
- 我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。比如，文本可以用 txt 格式表现，也可以用 HTML 表现。
- 如果客户端想要操作服务器，必须通过某种手段，让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。

通常认为：GET 用来获取资源，POST 用来新建资源（也可以用于更新资源），PUT 用来更新资源，DELETE 用来删除资源。

*GET 相对 POST 的优势是什么？*

- 请求中的 URL 可以被手动输入
- 请求中的 URL 可以被存在书签里，或者历史里。
- 请求中的 URL 可以被缓存。

### HTTPS

超文本传输安全协议（英语：*Hypertext Transfer Protocol Secure*，缩写：HTTPS，也被称为 HTTP over TLS，HTTP over SSL 或 HTTP Secure）是一种网络安全传输协议。在计算机网络上， HTTPS 经由超文本传输协议进行通讯，但利用 SSL/TLS 来对数据包进行加密。HTTPS 开发的主要目的，是提供对网络服务器的身份认证，保护交换数据的隐私与完整性。


### 状态码

 - 1xx 消息——请求已被服务器接收，继续处理
 - 2xx 成功——请求已成功被服务器接收、理解、并接受
 - 3xx 重定向——需要后续操作才能完成这一请求
 - 4xx 请求错误——请求含有词法错误或者无法被执行
 - 5xx 服务器错误——服务器在处理某个正确请求时发生错误
 - 200 OK 请求已成功，请求所希望的响应头或数据体将随此响应返回。
 - 403 Forbidden 服务器已经理解请求，但是拒绝执行它。
 - 404 Not Found 请求失败，请求所希望得到的资源未被在服务器上发现。
 - 500 Internal Server Error 服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。一般来说，这个问题都会在服务器的程序码出错时出现。

### 协议例子

客户端请求：

    GET / HTTP/1.1
    Host: www.google.com
 （末尾有一个空行。第一行指定方法、资源路径、协议版本；第二行是在1.1版里必带的一个 header 作用指定主机）   
    
服务器应答：

    HTTP/1.1 200 OK
    Content-Length: 3059
    Server: GWS/2.0
    Date: Sat, 11 Jan 2003 02:44:04 GMT
    Content-Type: text/html
    Cache-control: private
    Set-Cookie: PREF=ID=73d4aef52e57bae9:TM=1042253044:LM=1042253044:S=SMCc_HRPCQiqy
    X9j; expires=Sun, 17-Jan-2038 19:14:07 GMT; path=/; domain=.google.com
    Connection: keep-alive
    
（紧跟着一个空行，并且由 HTML 格式的文本组成了 Google 的主页）


  [1]: http://7xq3d5.com1.z0.glb.clouddn.com/6.png
  
  
  
参考资料：

[超文本传输协议](https://zh.wikipedia.org/wiki/%E8%B6%85%E6%96%87%E6%9C%AC%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)  
[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)  
[post 相比get 有很多优点，为什么现在的HTTP通信中大多数请求还是使用get？](https://www.zhihu.com/question/31640769?rf=37401322)