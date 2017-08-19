---
title: 初识JavaScript闭包
date: 2016-09-11 10:00:32
tags:
---
### 变量的作用域 

要理解闭包，首先必须理解 JavaScript 特殊的变量作用域。  
变量的作用域无非就是两种：全局变量和局部变量。  
JavaScript 中，函数内部可以直接读取全局变量。

       var n=999;
    　　function f1(){
    　　　　alert(n);
    　　}
    　　f1(); // 999

另一方面，在函数外部无法读取函数内的局部变量。（某个执行环境中的所有代码执行完毕后，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁）　　

       function f1(){
    　　　　var n=999;
    　　}
    　　alert(n); // error

这里有一个地方需要注意，函数内部声明变量的时候，一定要使用 var 命令。如果不用的话，你实际上声明了一个全局变量。

       function f1(){
    　　　　n=999;
    　　}
    　　f1();
    　　alert(n); // 999
    　　



**注意**：[(某些描述可能不够严谨)](https://www.zhihu.com/question/27712980)　访问 window 的属性创建的变量可以 delete，在全局作用域下直接声明的变量不可以 delete，因为使用 var 声明的变量的 configurable 属性是 false。

**词法作用域**

JavaScript 采用词法作用域，分析如下代码

	function foo() {
	    console.log( a ); 
	}
	
	function bar() {
	    var a = 3;
	    foo();
	}
	
	var a = 2;
	
	bar();

bar 被调用，bar 里面 foo 被调用，foo 函数需要查找变量 a，由于JavaScript 采用词法作用域，foo 被解析的时候是在全局作用域，所以 a 是全局作用域中的 2，而非 bar 里面的 a。假设 JavaScript 采用的是动态作用域，foo 是在 bar 中被调用的，所以 a 查找到了 bar 作用域里的 3。  

作为对照，动态作用域不关心它本身是怎样在哪里声明的，只关心它在哪里调用的。

相反，词法作用域关心的是函数在哪里声明的，动态作用域的概念和 JavaScript 中的 this 相同，this 也关心函数在哪里调用的。

### 闭包

    function makeFunc() {
      var name = "Mozilla";
      function displayName() {
        alert(name);
      }
      return displayName;
    }
    
    var myFunc = makeFunc();
    myFunc();

这段代码看起来别扭却能正常运行。通常，函数中的局部变量仅在函数的执行期间可用。一旦 makeFunc() 执行过后，我们会很合理的认为 name 变量将不再可用。虽然代码运行的没问题，但实际并不是这样的。

这个谜题的答案是 myFunc 变成一个闭包了。 闭包是一种特殊的对象。它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。在我们的例子中，myFunc 是一个闭包，由 displayName 函数和闭包创建时存在的 "Mozilla" 字符串形成。

维基百科中这样描述闭包：

>在计算机科学中，闭包（英语：Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

所谓自由变量是指：在A作用域中使用的变量 x，却没有在 A 作用域中声明（即在其他作用域中声明的），对于 A 作用域来说，x 就是一个自由变量。

MDN 中定义：

>闭包是指那些能够访问独立(自由)变量的函数 (变量在本地使用，但定义在一个封闭的作用域中)。换句话说，这些函数可以“记忆”它被创建时候的环境。

Douglas Crockford 说：
>之所以可能通过这种方式在 JavaScript 种实现公有，私有，特权变量正是因为闭包，闭包是指在 JavaScript 中，内部函数总是可以访问其所在的外部函数中声明的参数和变量，即使在其外部函数被返回了之后。

### 闭包的用途

- 读取函数内部的变量
- 让这些变量的值始终保持在内存中


### 其它
- JavaScript 闭包的本质源自两点，**词法作用域**和**函数当作值传递**。  
**词法作用域**：就是内部函数可以访问函数外面的变量。  
**函数当作值传递**：就是可以把函数当作一个值来赋值，当作参数传给别的函数，也可以把函数当作一个值 return。  
- 闭包的形成很简单，在执行过程完毕后，返回函数，或者将函数得以保留下来，即形成闭包。
- JavaScript 中的函数运行在它们被定义（声明）的作用域里，而不是它们被执行（调用）的作用域里。
- 闭包与 Java 内部类有相似之处

**注意**：因为闭包会使得函数中变量都保存在内存中，因此不能滥用闭包，否则会造成内存溢出。

### Java与闭包

**类和对象**

Java 主流语法不允许直接的函数嵌套，但是 Java 中真的不存在闭包吗？正好相反，Java 到处都是闭包。因为 Java 的对象其实就是一个闭包。其实无论是闭包还是对象，都是数据封装的手段。

	class Add{
	    private int x=1;
	    public int add(){
		   int y=2;
	    	return x+y;
	    }
	}

看上去`x`在函数`add()`的作用域外面，但是通过 Add 类实例化的过程，变量`x`和数值1之间已经绑定了，而且和函数`add()`也已经打包在一起。`add()`函数其实是透过 this 关键字来访问对象的成员字段的，即`this.x`

**内部类是闭包**

Java 中的内部类就是一个典型的闭包结构。代码如下

	public class Outer {
	    private class Inner{
	        private int x=1;
	        public int innerAdd(){
	            return x+y;
	        }
	    }
	    private int y=2;
	}

下图画的就是上面代码的结构。内部类（Inner Class）通过包含一个指向外部类的引用，做到自由访问外部环境类的所有字段，变相把环境中的自由变量封装到函数里，形成一个闭包。

![java](http://7xq3d5.com1.z0.glb.clouddn.com/c_java_inner_class.png)

**为什么匿名内部类的参数要用 final 修饰？**

	class Outer {
	    public AnnoInner getAnnoInner(final int x) {
	        final int y = 100;
	        return new AnnoInner() {
	            @Override
	            public int add() {
	                int z = 101;
	                return x + y + z;
	            }
	        };
	    }
	
	    interface AnnoInner {
	        int add();
	    }
	}

我们看到，`add()`函数直接使用了`x`和`y`两个自由变量，这就说明`getAnnoInner()`方法已经对内部类 AnnoInner 构成了一个闭包。
然后，`x`和`y`都必须用 final 修饰，无法更改。这是为什么呢？因为这里 Java 编译器支持了闭包，但支持地不完整。说支持了闭包，是因为编译器编译的时候其实悄悄对函数做了手脚，偷偷把外部环境方法的`x`和`y`局部变量，拷贝了一份到匿名内部类里。如下面的代码所示。

	class Outer {
	    public AnnoInner getAnnoInner(final int x) {
	        final int y = 100;
	        return new AnnoInner() {
	            @Override
	            public int add() {
	            	   //编译器相当于拷贝了外部自由变量x的一个副本到匿名内部类里
	                int copyX = x; 
	                //编译器相当于拷贝了外部自由变量y的一个副本到匿名内部类里
	                int copyY = y;
	                int z = 101;
	                return x + y + z;
	            }
	        };
	    }
	
	    interface AnnoInner {
	        int add();
	    }
	}


>为什么仅仅针对方法中的参数限制final，而访问外部类的属性就可以随意?

因为每个内部类的实例都隐藏了一个指向外部类实例的引用。Java 只是没有显式的写出来而已。内部类访问外部类成员都是通过这个引用。之所以能有这个引用，是因为两者都是实例，都有自己的内存空间。而匿名内部类的外围环境函数只是一个函数，执行完之后，也就是**匿名内部类诞生（初始化）完成的那一刻，它的生存周期就结束了。函数内部的局部变量（包括函数的参数）也就跟着被销毁了**。所以产生出来的内部类根本无法像保留外部类的引用那样保留外围环境函数的引用。所以只能退而求其次，只保留一份局部变量的拷贝值。

	
所以，Java 编译器实现的只是**值捕获 capture-by-value**，并没有实现**引用捕获 capture-by-reference**，只有后者才能保持匿名内部类和外部环境局部变量保持同步。既然内外不能同步，那 Java 就干脆不允许更改外围的局部变量。


参考资料：  
[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)  
[wikipedia: 闭包 (计算机科学)](https://zh.wikipedia.org/w/index.php?title=%E9%97%AD%E5%8C%85_%28%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6%29&variant=zh-cn)  
[MDN: 闭包](https://developer.mozilla.org/cn/docs/Web/JavaScript/Closures)  
[Private Members in JavaScript](http://www.crockford.com/javascript/private.html)  
[JavaScript 里的闭包是什么？应用场景有哪些？](https://www.zhihu.com/question/19554716)  
[理解JavaScript的闭包](http://wwsun.github.io/posts/javascript-closure.html)  
[《JavaScript高级程序设计》 ch4 ch7]()  
[You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS/tree/master/scope%20%26%20closures)  
[JVM的规范中允许编程语言语义中创建闭包(closure)吗？](https://www.zhihu.com/question/27416568/answer/36565794)
