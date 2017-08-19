---
title: JavaScript中的作用域和执行上下文
date: 2016-09-07 17:18:32
tags:
---

### 执行上下文(执行环境)(execution context)
1.JavaScript 是一个**单线程语言**（[如何实现异步编程？](https://haoduoyu.cc/2016/12/05/%E6%B5%85%E8%B0%88JavaScript%E4%B8%AD%E7%9A%84%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/)），意味着同一时间只能执行一个任务。当 JavaScript 解释器初始化执行代码时， 它首先默认进入全局执行环境（execution context），从此刻开始，函数的每次调用都会创建一个新的执行环境。 JavaScript 高级程序设计中这样定义执行环境：**执行环境定义了变量或函数有权访问的其它数据，决定了它们各自的行为。**每个执行环境都有一个与之关联的变量对象(variable object)，环境中定义的所有变量和函数都保存在这个对象中。我们无法访问这个对象，但解析器在处理数据时会在后台使用它。  
2.**执行环境**非常重要，有下面几种不同的情况：

 - 全局代码——你的代码首次执行的默认环境。
 - 函数代码——每当进入一个函数内部。
 - Eval 代码——eval 内部的文本被执行时。

![执行上下文](http://yanhaijing.com/blog/136.jpg)

我们有一个被紫色边框圈起来的全局上下文和三个分别被绿色，蓝色和橘色框起来的不同函数上下文。只有全局上下文（的变量）能被其他任何上下文访问。

执行环境更偏向于作用域的作用，而不是上下文（Context）。

3.每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境就会被推入一个**环境栈中（execution stack）**。在函数执行完后，栈将其环境弹出， 把控制权返回给之前的执行环境。ECMAScript 程序中的执行流正是由这个便利的机制控制着。

关于执行栈（调用栈），有5个关键点：

- 单线程。
- 同步执行。
- 一个全局上下文。
- 无限制函数上下文。
- 每次函数被调用创建新的执行上下文，包括调用自己。

4.在 JavaScript 解释器内部，每次调用执行上下文，分为两个阶段：
 
**创建阶段**（当函数被调用，但未执行任何其内部代码之前） 
 
 - 解析器首先会创建一个变量对象（variable object，也称为活动对象 activation object）， 它由定义在执行环境中的变量、函数声明、和参数组成。  
 - 作用域链会被初始化。  
 - this的值也会被最终确定。   
 
**执行阶段**

 - 代码被解释执行。

可以将每个执行上下文抽象为一个对象并有三个属性：

	executionContextObj = {
	    scopeChain: { /* 变量对象（variableObject）+ 所有父执行上下文的变量对象*/ }, 
	    variableObject: { /*函数 arguments/参数，内部变量和函数声明 */ }, 
	    this: {} 
	}


每个执行环境都有一个与之关联的变量对象（variable object），环境中定义的所有变量和函数都保存在这个对象中。我们无法手动访问这个对象，只有解析器才能访问它。

### 上下文(Context)和作用域(Scope)
1.根本上来说，作用域是基于函数的，而上下文是基于对象的。  
2.作用域涉及到所被调用函数中的变量访问，并且不同的调用场景是不一样的。  
3.上下文始终为 this 关键字的值， 它是拥有（控制）当前所执行代码的对象的引用。  

### this上下文
上下文通常取决于函数是如何被调用的。当一个函数被作为**对象中的一个方法**被调用的时候，this 为调用该方法的对象：


	var obj = {
	    foo: function(){
	        alert(this === obj);    
	    }
	};
	obj.foo(); // true

这个准则也适用于当调用函数时使用 new 操作符来创建对象的实例的情况下。在这种情况下，在函数的作用域内部 this 的值被设置为新创建的实例：

	function foo(){
	    alert(this);
	}
	new foo() // foo
	foo() // window


当**直接调用**一个函数时，this 默认情况下是全局上下文，在浏览器中它指向 window 对象。  

需要注意的是，ES5 引入了严格模式的概念， 如果启用了严格模式，此时上下文默认为undefined。

**注意**：ES6 箭头函数不仅仅是让代码变得简洁，函数体内的this 对象，就是定义时所在的对象，而不是使用时所在的对象，也叫做**词法作用域**的 this 值

	var obj = {
	    name: 'name',
	    foo: function () {
	        console.log(this); // Object {name: "name"}
	        setTimeout(function () {
	            console.log(this);  // Window //属于直接调用
	        }, 1000);
	    },
	    foo2: function () {
	        console.log(this); // Object {name: "name"}
	        setTimeout(() => {
	            console.log(this);  // Object {name: "name"}
	        }, 2000);
	    }
	}
	obj.foo();
	obj.foo2();


`apply`、`call`、`bind`三者都可以用来改变函数 this 对象的指向：[JavaScript call() , apply() , bind()方法
](http://www.jianshu.com/p/92d3e835764b)

### 变量作用域

1.任何被定义的全局变量需要在函数体的外部被声明，并且存活于整个运行时（runtime），并且在任何作用域中都可以被访问到。   
2.在 ES6 之前，局部变量存在于函数体中，并且函数的每次调用都有不同的作用域。   
3.在 ES6 之前，JavaScript不支持块级作用域，这意味着在 if 语句、switch 语句、for 循环、while 循环中无法支持块级作用域。  
4.从 ES6 开始，你可以通过 let 关键字来定义变量，它修正了 var 关键字的缺点，能够让你像 Java 语言那样定义变量，并且支持块级作用域。  
ES6 之前，我们使用 var 关键字定义变量：  
```
function func() {
  if (true) {
    var tmp = 123;
  }
  console.log(tmp); // 123
}
```
之所以能够访问，是因为 var 关键字声明的变量有一个变量提升的过程。而在 ES6 场景，推荐使用 let 关键字定义变量：  
```
function func() {
  if (true) {
    let tmp = 123;
  }
  console.log(tmp); // ReferenceError: tmp is not defined
}
```

### 作用域链(The Scope Chain)

作用域链可以保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。  
每次一个新的执行上下文被创建时，它都被添加到了作用域链（有时它也被称为执行栈或者调用栈）的顶部。浏览器总是执行当前位于作用域链顶部的执行上下文。一旦执行完成，当前执行上下文就会被从栈顶移除，并将控制权交给它下面的执行上下文。例如：
```
function first(){
    second();
    function second(){
        third();
        function third(){
            fourth();
            function fourth(){
                // do something
            }
        }
    }   
}
first();
```
运行前面的代码将会导致嵌套的函数被从上倒下执行直到 fourth 函数，此时作用域链从上到下为： fourth, third, second, first, global。fourth 函数能够访问全局变量和任何在 first,second 和 third 函数中定义的变量。一旦 fourth 函数执行完成，它将被从作用域链顶端移除，执行权将返回到 thrid 函数。这一过程持续进行，直到所有的代码完成执行。

不同执行上下文之间的变量命名冲突通过攀爬作用域链解决，从局部直到全局。这意味着拥有相同名字并位于作用域链更上方的的局部变量会被优先获取。

简单的说，每次你试图访问函数执行上下文中的变量时，查找进程总是从自己的变量对象开始。如果在自己的变量对象中没发现要查找的变量，将继续搜索作用域链。它将攀爬作用域链检查每一个执行上下文的变量对象，去寻找和变量名称匹配的值。


参考资料：
[Understanding Scope and Context in JavaScript](http://modernweb.com/2013/08/26/understanding-scope-and-context-in-javascript/)  
[认识javascript中的作用域和上下文](http://yanhaijing.com/javascript/2013/08/30/understanding-scope-and-context-in-javascript/)  
[了解JavaScript的执行上下文](http://yanhaijing.com/javascript/2014/04/29/what-is-the-execution-context-in-javascript/)  
[理解JavaScript中的作用域和上下文](http://wwsun.github.io/posts/scope-and-context-in-javascript.html)




