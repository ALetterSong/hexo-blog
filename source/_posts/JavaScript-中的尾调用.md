---
title: JavaScript 中的尾调用
date: 2017-02-17 15:05:09
tags:
---


有一个著名的 fibonacci 递归算法，

	function fibonacci(n) {
	        return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
	    }


当在浏览器运行这个函数时，会出现一些意想不到的结果，当我们运行`fibonacci(10)`时，程序可以正常运行，但如果把参数改大点，运行`fibonacci(50)`，浏览器就会出现假死，这还得从尾调用说起。

![](http://7xq3d5.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-17%20%E4%B8%8A%E5%8D%889.46.31.png?imageView2/2/w/380)

## 尾调用

尾调用(Tail Call)是函数式编程里的一个重要概念，它是指某个函数的最后一步是调用另一个函数。

	function f(x){
	  return g(x);
	}

但以下情况都不属于尾调用	

	// 调用函数g之后，还有赋值操作
	function f(x){
	  let y = g(x);
	  return y;
	}
	
	// 属于调用后还有操作，即使写在一行内
	function f(x){
	  return g(x) + 1;
	}
	
	// 等同于return undefined
	function f(x){
	  g(x);
	}	
	
函数 m 和 n 都属于尾调用，因为它们都是函数 f 的最后一步操作

	function f(x) {
	  if (x > 0) {
	    return m(x)
	  }
	  return n(x);
	}	
		
## 尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。而开头所说的Fibonacci 数列就用到了递归。

	0, 1, 1, 2, 3, 5, 8, 13, 21, ...  
	
fibonacci 函数
	
		function fibonacci(n) {
	        return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
	    }
	    
以 n = 5 为例，函数调用栈会像这样展开

	[fibonacci(5)]
	[fibonacci(4) + fibonacci(3)]
	[(fibonacci(3) + fibonacci(2)) + (fibonacci(2) + fibonacci(1))]
	[((fibonacci(2) + fibonacci(1)) + (fibonacci(1) + fibonacci(0))) + ((fibonacci(1) + fibonacci(0)) + fibonacci(1))]
	[fibonacci(1) + fibonacci(0) + fibonacci(1) + fibonacci(1) + fibonacci(0) + fibonacci(1) + fibonacci(0) + fibonacci(1)]
	[1 + 0 + 1 + 1 + 0 + 1 + 0 + 1]
	5  	    
	
才到第 5 项调用栈长度就有 8 了，一些复杂点的递归稍不注意就会超出限度，同时也会消耗大量内存。而如果用尾递归的方式来优化这个过程，就可以避免这个问题，用尾递归来求 Fibonacci 数列的值可以写成这样：

	function fibonacci2 (n , a = 0 , b = 1) {
	  if( n === 0 ) {return a};
	
	  return fibonacci2 (n - 1, b, a + b);
	}	
	
n 用来记录递归剩余的次数，假如 a 是第 m 个数，b 就是第 m 个数和前一个数的和，写成调用栈就是：

	fibonacci2(5) === fibonacci2(5, 0, 1)  
	fibonacci2(4, 1, 1)  
	fibonacci2(3, 1, 2)  
	fibonacci2(2, 2, 3)  
	fibonacci2(1, 3, 5)  
	fibonacci2(0, 5, 8) => return 5  	
	
传入较大的参数也不会发生栈溢出(stack overflow)了：
	
![](http://7xq3d5.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-02-17%20%E4%B8%8A%E5%8D%8810.43.11.png?imageView2/2/w/380)

## 尾调用优化

>尾调用在没有进行任何优化的时候和其他的递归方式一样，该产生的调用栈一样会产生，一样会有栈溢出的危险。

我们知道，函数调用会在内存形成一个"调用记录"，又称"调用帧"（call frame），保存调用位置和内部变量等信息。

如果在函数 A 的内部调用函数 B，那么在 A 的调用帧上方，还会形成一个 B 的调用帧。等到 B 运行结束，将结果返回到 A，B 的调用帧才会消失。如果函数 B 内部还调用函数 C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个"调用栈"（call stack）。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

![](http://image.beekka.com/blog/2015/bg2015041002.png)


	function f() {
	  let m = 1;
	  let n = 2;
	  return g(m + n);
	}
	f();
	
	// 等同于
	function f() {
	  return g(3);
	}
	f();
	
	// 等同于
	g(3);
	
上面代码中，如果函数 g 不是尾调用，函数 f 就需要保存内部变量 m 和 n 的值、g 的调用位置等信息。但由于调用 g 之后，函数 f 就结束了，所以执行到最后一步，完全可以删除 f(x) 的调用帧，只保留 g(3) 的调用帧。

这就叫做“尾调用优化”（Tail call optimization），即只保留内层函数的调用帧。调用帧只有一项将大大节省内存。这就是“尾调用优化”的意义。	

## 递归优化

在 JavaScript 的递归调用中，JavaScript 引擎将为每次递归开辟一段内存用以储存递归截止前的数据，这些内存的数据结构以“栈”的形式存储，这种方式开销非常大，并且一般浏览器可用的内存非常有限。

### 改为循环

	 function fibonacciLoop(n) {
	        var a = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : 0;
	        var b = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : 1;
	        while (n--) {
	            [a, b] = [b, a + b]
	        }
	        return a
	    }

### 非侵入式的尾递归

一般来说，尾调用优化(Tail Call Optimization, TCO)是可选的。然而，在函数编程语言中，语言标准通常会要求虚拟机实现尾调用消除，这让程序员可以用递归替换循环而不丧失性能。比如 ES6 的严格模式下会开启尾调用优化。下面是一段尾递归优化函数：


	function tailCallOptimize(f) {  
	  let value,
	      active = false
	  const accumulated = []
	  return function accumulator() {
	    accumulated.push(arguments)
	    if (!active) {
	      active = true
	      while (accumulated.length) {
	        value = f.apply(this, accumulated.shift())
	      }
	      active = false
	      return value
	    }
	  }
	}
	

参看资料：

[尾调用优化](http://es6.ruanyifeng.com/#docs/function#尾调用优化)  
[JavaScript 中的尾调用](https://fe.ele.me/javascript-zhong-de-wei-diao-yong/)  
[JS的递归与TCO尾调用优化](https://segmentfault.com/a/1190000004018047)  