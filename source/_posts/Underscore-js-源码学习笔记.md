---
title: Underscore.js 源码学习笔记
date: 2017-03-07 15:25:03
tags:
---

> 之前阅读过 [Zepto](https://github.com/madrobby/zepto/) 源码，感觉直接去读源代码有些难度，但如果结合着源码分析类文章阅读则会轻松很多，而且有利于抓住重点。下面就是读 [Underscore.js 源码解读](https://github.com/hanzichi/underscore-analysis) 的笔记。

### 「void 0」代替「undefined」

undefined 并不是保留词（reserved word），它只是全局对象的一个属性，在低版本 IE 中能被重写。

事实上，undefined 在 ES5 中已经是全局对象的一个只读（read-only）属性了，它不能被重写。但是在局部作用域中，还是可以被重写的。

void 运算符能对给定的表达式进行求值，然后返回 undefined [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void)

### 类型判断 

	_.isArray = nativeIsArray || function(obj) {
	    return toString.call(obj) === '[object Array]';
	  };
	  
	// Add some isType methods: isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError.
	// 其他类型判断
	_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
	  _['is' + name] = function(obj) {
	    return toString.call(obj) === '[object ' + name + ']';
	  };
	});
	
使用 Object 的原型方法 toString()来获取 obj 的字符串表示，形式是 [object class]	

	  // Is the given value `NaN`? (NaN is the only number which does not equal itself).
	  _.isNaN = function(obj) {
	    return _.isNumber(obj) && obj !== +obj;
	  };	
*原生 isNaN 方法存在怪异行为：如果 isNaN 函数的参数不是 Number 类型,  isNaN() 会首先尝试将这个参数转换为数值，然后才会对转换后的结果是否是 NaN 进行判断。*

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/isNaN)	  	                    
### extend, extendOwn 的内部方法 createAssigner  

	    // for-in 会遍历出一个对象从其原型链上继承到的可枚举属性
	    var allKeys = function (obj) {
	        var keys = [];
	        for (var key in obj) keys.push(key);
	        return keys;
	    };
	
	    var keys = Object.keys
	
	    var createAssigner = function (keysFunc, undefinedOnly) {
	        return function (obj) {
	            var length = arguments.length;
	            if (length < 2 || obj == null) return obj;
	            for (var index = 1; index < length; index++) {
	                var source = arguments[index],
	                    keys = keysFunc(source),
	                    l = keys.length;
	                for (var i = 0; i < l; i++) {
	                    var key = keys[i];
	                     // undefinedOnly 参数为 true, 即 !undefinedOnly 为 false
                        // 那么当且仅当 obj[key] 为 undefined 时才覆盖
                        // 即如果有相同的 key 值，取最早出现的 value 值
	                    if (!undefinedOnly || obj[key] === void 0) obj[key] = source[key];
	                }
	            }
	            return obj;
	        };
	    };
	
	    var a = {a: 'hello'}
	    function Super(name) {
	        this.name = name;
	    }
	    Super.prototype.superman = 'superman';
	
	    var b = new Super('world')
	    var c = {c: 'hello'}
	
	    // extend 复制source对象中的所有属性覆盖到destination对象上
	    console.log(createAssigner(allKeys)(a, b)) // {a: "hello", name: "world", superman: "superman"}
	    // extendOwn 只复制自己的属性覆盖到目标对象
	    console.log(createAssigner(keys)(c, b)) // {c: "hello", name: "world"}
	    
createAssigner 返回了一个函数，这个返回的函数引用了外面的一个变量，这是一个典型的闭包。这有利于设计用途类似的 API。
	    
### 比较两个元素是否相等 

#### 普通类型判断

        // 0 和 -0 被认为是不相等, 1/0 === 1/-0 => false
        if (a === b) return a !== 0 || 1 / a === 1 / b;
        // 如果 a 和 b 有一个为 null（或者 undefined）
        if (a == null || b == null) return a === b;
        // 如果 a 和 b 类型不相同，则返回 false
        var className = toString.call(a);
        if (className !== toString.call(b)) return false;
        switch (className) {
            case '[object RegExp]':
            case '[object String]':
                // 转为 String 类型进行比较
                return '' + a === '' + b;
            case '[object Number]':
                // NaN 与 NaN 相等
                if (+a !== +a) return +b !== +b;
                return +a === 0 ? 1 / +a === 1 / b : +a === +b;
            case '[object Date]':
            case '[object Boolean]':
                return +a === +b;
        }
        
#### Array 和 Object 类型判断 

	   function isEquivalent(a, b) {
	        // Create arrays of property names
	        var aProps = Object.getOwnPropertyNames(a);
	        var bProps = Object.getOwnPropertyNames(b);
	
	        // If number of properties is different,
	        // objects are not equivalent
	        if (aProps.length != bProps.length) {
	            return false;
	        }
	
	        for (var i = 0; i < aProps.length; i++) {
	            var propName = aProps[i];
	
	            // If values of same property are not equal,
	            // objects are not equivalent
	            if (a[propName] !== b[propName]) {
	                return false;
	            }
	        }
	
	        // If we made it this far, objects
	        // are considered equivalent
	        return true;
	    }
	
	    function arraysEqual(a, b) {
	        if (a === b) return true;
	        if (a == null || b == null) return false;
	        if (a.length != b.length) return false;
	
	        // If you don't care about the order of the elements inside
	        // the array, you should sort both arrays here.
	
	        for (var i = 0; i < a.length; ++i) {
	            if (a[i] !== b[i]) return false;
	        }
	        return true;
	    }
  

### 函数去抖和函数节流 

通过 RxMarbles(珠宝图) 可以形象地理解函数去抖和函数节流

![debounce](http://upload-images.jianshu.io/upload_images/257925-15e0dbdec48f7fb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![throttle](http://upload-images.jianshu.io/upload_images/257925-4fd64092b0c88e9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

throttle 和 debounce 的应用场景:

(1) 按一个按钮发送 AJAX：给 click 加了 **debounce** 后就算用户不停地点这个按钮，也只会最终发送一次；如果是 throttle 就会间隔发送几次

(2) 监听滚动事件判断是否到页面底部自动加载更多：给 scroll 加了 debounce 后，只有用户停止滚动后，才会判断是否到了页面底部；
如果是 **throttle** 的话，只要页面滚动就会间隔一段时间判断一次

#### 函数去抖 

函数去抖就是对于一定时间内的连续的函数调用，只让其执行一次。核心思想是重复添加定时器。

简单实现：

	function debounce(func, wait) {
	        // 闭包缓存 timeout
	        var timeout, result;
	        // 返回函数
	        return function () {
	            var context = this, args = arguments;
	            clearTimeout(timeout);
	            timeout = setTimeout(function () {
	                timeout = null;
	                result = func.apply(context, args);
	            }, wait);
	            return result;
	        };
	    }

immediate 为 true， debounce会在 wait 时间间隔的开始调用这个函数 	

	_.debounce = function(func, wait, immediate) {
	    var timeout, args, context, timestamp, result;
	
	    var later = function() {
	      var last = _.now() - timestamp;
	
	      if (last < wait && last >= 0) {
	        timeout = setTimeout(later, wait - last);
	      } else {
	        timeout = null;
	        if (!immediate) {
	          result = func.apply(context, args);
	          if (!timeout) context = args = null;
	        }
	      }
	    };
	
	    return function() {
	      context = this;
	      args = arguments;
	      timestamp = _.now();
	      var callNow = immediate && !timeout;
	      if (!timeout) timeout = setTimeout(later, wait);
	      if (callNow) {
	        result = func.apply(context, args);
	        context = args = null;
	      }
	
	      return result;
	    };
	  };    
	function print() {
	  console.log('hello world');
	}
	
	window.onscroll = _.debounce(print, 1000);
	    
#### 函数节流 

函数节流是指一个函数在一定间隔内调用，目的是让一个函数不要执行得太频繁。

	function throttle(func, wait) {
	        var timeout, result;
	        return function () {
	            var context = this, args = arguments;
	            // 如果不存在定时器，则设置定时器
	            if (!timeout)
	                timeout = setTimeout(function () {
	                    // 执行完成清空定时器
	                    timeout = null;
	                    result = func.apply(context, args);
	                }, wait);
	            return result;
	        };
	    }

### 函数记忆 

Memoization（记忆化） 原理非常简单，就是把函数的每次执行结果都放入一个散列表中，在接下来的执行中，在散列表中查找是否已经有相应执行过的值，如果有，直接返回该值，没有才真正执行函数体的求值部分。很明显，找值，尤其是在散列中找值，比执行函数快多了。

	var memoize = function (func) {
	        var cache = {};
	        return function (key) {
	            if (!cache[key])
	                cache[key] = func.apply(this, arguments);
	            return cache[key];
	        }
	    }
	
	    var fibonacci = memoize(function (n) {
	        return n < 2 ? n : fibonacci(n - 2) + fibonacci(n - 1);
	    });

### NaN
	
	    var isNaN1 = function (obj) {
	        return obj !== obj;//错误：new Number(NaN) => false
	    };
	    var isNaN2 = function (obj) {
	        return _.isNumber(obj) && obj !== +obj;//错误：new Number(0) => true
	    };
	    var isNaN3 = function (obj) {
	        return _.isNumber(obj) && isNaN(obj);//正确：new Number(NaN) => true
	    };
	
对于`NaN`的判断，如果只针对`Number`类型，用 underscore 最新版的 `_.isNaN3` 判断完全没有问题，
或者用 ES6 的 `Number.isNaN`，两者的区别就在于一个 `new Number(NaN)`

	Number.isNaN(new Number(NaN)) =>  false


### Bind 

bind()方法会创建一个新函数。当这个新函数被调用时，bind()的第一个参数将作为它运行时的 this, 之后的一序列参数将会在传递的实参前传入作为它的参数。

#### 偏函数与柯里化

函数柯里化的本质是，可以在调用一个函数的时候传入更少的参数，而这个函数会返回另外一个函数并且能够接收其他参数。是一种实现**多参数函数**的方法。

	function add(x){
	    return function(y){
	        return x + y;
	    }
	}
	var inc = add(1)
	var dev = add(-1)
	inc(1) // 2
	dev(1) // 0
	
偏函数应用到了 bind ，他解决这样的问题：如果我们有函数是多个参数的，我们希望能**固定其中某几个参数的值**。

	function list() {
	  return Array.prototype.slice.call(arguments);
	}
	
	var list1 = list(1, 2, 3); // [1, 2, 3]
	
	// Create a function with a preset leading argument
	var leadingThirtysevenList = list.bind(undefined, 37);
	
	var list2 = leadingThirtysevenList(); // [37]
	var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]
		    
### 类数组  

ArrayLike 类数组需满足两点要求：

- 有 length 属性
- length 为非负 Number 类型     

``` 
	 var isArrayLike = function(collection) {
	    var length = getLength(collection);
	    return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
	  };
```
ArrayLike to Array:

	function fn() {
	  var arr = [].slice.call(arguments);
	  arr.push(4); // arr -> [1, 2, 3, 4]
	}	 
	 
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)	

ES6:

	var str = "helloworld";
	var arr = Array.from(str); 
	// ["h", "e", "l", "l", "o", "w", "o", "r", "l", "d"]
	
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)

### 数组去重 

双重循环法：

	 function unique(arr) {
        var res = [];
        var isRepeat;
        for (var i = 0; i < arr.length; i++) {
            isRepeat = false;
            for (var j = i + 1; j < arr.length; j++) {
                if (arr[i] === arr[j]) {
                    isRepeat = true;
                    break;
                }
            }
            if (!isRepeat) {
                res.push(arr[i]);
            }
        }
        return res;
    }
    
ES5:

	function unique(arr) {
	        return arr.filter(function (item, index, array) {
	            // indexOf()方法返回给定元素能找在数组中找到的第一个索引值
	            return array.indexOf(item) === index;
	        });
	    }  
	    
ES6:

	function unique(arr) {
	        return Array.from(new Set(arr));
	    }	   
  
### 数组展开

双重循环法：

	function flatten(arr) {
	     var result = [];
	     for (var i = 0; i < arr.length; i++) {
	         for (var j = 0; j < arr[i].length; j++) {
	             result.push(arr[i][j]);
	         }
	     }
	     return result;
	}
	    
apply + concat：

	function flatten(arr) {
	     return Array.prototype.concat.apply([], arr);
	}	
	
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)	
    
多维数组：

	function flatten(arr) {
	   var tmp = arr;
	   var result = arr;
	   while(tmp instanceof Array) {
	      result = Array.prototype.concat.apply([], result);
	           tmp = tmp[0];
	      }
	   return result;
	}	
	
	flatten([[[1, 2], [1, 2, 3]], [1, 2]]) // [1, 2, 1, 2, 3, 1, 2]  


### 数组乱序 

splice 方法：

	function shuffle(a) {
	  var b = [];
	
	  while (a.length) {
	    //https://coderwall.com/p/9b6ksa/is-faster-than-math-floor
	    var index = ~~(Math.random() * a.length);
	    b.push(a[index]);
	    a.splice(index, 1);
	  }
	
	  return b;
	}

sort 方法：	

	function shuffle(a) {
	        // concat 复制原数组
	        return a.concat().sort(function (a, b) {
	            return Math.random() - 0.5;
	        });
	    }

**注意**：Array.prototype.sort 不能做到完全随机，这取决于排序算法的实现
	    
Fisher–Yates Shuffle:

	    // 我们每一次循环从前 len - i 个元素里随机一个位置，将这个元素和第 len - i 个元素进行交换，迭代直到 i = len - 1 为止
	    // 比如长度为5的数组，第一次循环从前五个元素中随机一个和第五个元素交换，
	    // 第二次从前四个元素中随机一个和第四个元素交换
	    function shuffle(a) {
	        var arr = a.concat()
	        var len = arr.length;
	        for (var i = 0; i < len - 1; i++) {
	            var idx = ~~(Math.random() * (len - i));
	            var temp = arr[idx];
	            arr[idx] = arr[len - i - 1];
	            arr[len - i - 1] = temp;
	        }
	        return arr;
	    }   
	

### 模板引擎 

#### 后端 MVC

后端 MVC 模式中，一般从 Model 层中读取数据，然后将数据传到 View 层渲染（渲染成 HTML 文件），而 View 层，一般都会用到模板引擎。比如PHP 的 smarty 模板引擎。

	<div>
	  <ul class="well nav nav-list" style="height:95%;">
	    {{foreach from=$pageArray.result item=leftMenu key=key name=leftMenu}}
	      <li class="nav-header">{{$key}}</li>
	      {{foreach from=$leftMenu key=key2 item=item2}}
	        <li><a target="main" href='{{$item2}}'>{{$key2}}</a></li>
	      {{/foreach}}
	    {{/foreach}}
	  </ul>
	</div>
	
#### 前端模板

假设接口数据如下：

	[{name: "apple"}, {name: "orange"}, {name: "peach"}]
	
渲染后的页面如下：

	<div>
	  <ul class="list">
	    <li>apple</li>
	    <li>orange</li>
	    <li class="last-item">peach</li>
	  </ul>
	</div>	
	
前端模板引擎出现之前，我们一般会这么做：

	<div></div>
	<script>
	// 假设接口数据
	var data = [{name: "apple"}, {name: "orange"}, {name: "peach"}];
	
	var str = "";
	str += '<ul class="list">';
	
	for (var i = 0, len = data.length; i < len; i++) {
	  if (i !== len - 1)
	    str += "<li>" + data[i].name + "</li>";
	  else
	    str += '<li class="last-item">'  + data[i].name + "</li>";
	}
	
	str += "</ul>";
	document.querySelector("div").innerHTML = str;
	</script>	
	
将 HTML 代码（View 层）和 JS 代码（Controller 层）混杂在了一起很容易出错也不利于维护。所以前端模板引擎出现了：

	<div></div>
	<script src="//cdn.bootcss.com/underscore.js/1.8.3/underscore.js"></script>
	<script type="text/template" id="tpl">
	  <ul class="list">
	    <%_.each(obj, function(e, i, a){%>
	      <% if (i === a.length - 1) %>
	        <li class="last-item"><%=e.name%></li>
	      <% else %>
	        <li><%=e.name%></li>
	    <%})%>
	  </ul>
	</script>
	
	<script>
	// 模拟数据
	var data = [{name: "apple"}, {name: "orange"}, {name: "peach"}];
	
	var compiled = _.template(document.getElementById("tpl").innerHTML);
	var str = compiled(data);
	document.querySelector("div").innerHTML = str;
	</script>
	
#### Node 中间层

模板引擎虽然降低了耦合度，但是却不利于 SEO。我们可以让 Node 作为中间层。简单地说就是让一门后台语言提供接口，Node 中间层用模板引擎来渲染页面，使得页面直出。不失为一种比较好的解决方案。
         
### Object.create() Polyfill

Object.create() 方法创建一个拥有指定原型和若干个指定属性的对象。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

        var create = (function () {
        //为了节省内存，使用一个共享的构造器
        function Temp() {
        }

        // 使用 Object.prototype.hasOwnProperty 更安全的引用
        var hasOwn = Object.prototype.hasOwnProperty;

        return function (O) {
            // 1. 如果 O 不是 Object 或 null，抛出一个 TypeError 异常。
            if (typeof O != 'object') {
                throw TypeError('Object prototype may only be an Object or null');
            }

            // 2. 使创建的一个新的对象为 obj ，就和通过
            //    new Object() 表达式创建一个新对象一样，
            //    Object是标准内置的构造器名
            // 3. 设置 obj 的内部属性 [[Prototype]] 为 O。
            Temp.prototype = O;
            var obj = new Temp();
            Temp.prototype = null; // 不要保持一个 O 的杂散引用（a stray reference）...

            // 4. 如果存在参数 Properties ，而不是 undefined ，
            //    那么就把参数的自身属性添加到 obj 上，就像调用
            //    携带obj ，Properties两个参数的标准内置函数
            //    Object.defineProperties() 一样。
            if (arguments.length > 1) {
                // Object.defineProperties does ToObject on its first argument.
                var Properties = Object(arguments[1]);
                for (var prop in Properties) {
                    if (hasOwn.call(Properties, prop)) {
                        obj[prop] = Properties[prop];
                    }
                }
            }

            // 5. 返回 obj
            return obj;
        };
    })();
 
 _.create 方法思路大致如此： 
   
	var baseCreate = function(prototype) {
	    if (!_.isObject(prototype)) return {};
	    if (nativeCreate) return nativeCreate(prototype);
	    Ctor.prototype = prototype;
	    var result = new Ctor;
	    Ctor.prototype = null;
	    return result;
	  };
	  
	_.create = function(prototype, props) {
	    var result = baseCreate(prototype);
	    if (props) _.extendOwn(result, props);
	    return result;
	  };

    
### Array.prototype.findIndex() Polyfill

findIndex() 方法返回数组中满足提供的测试函数的第一个元素的索引。否则返回-1。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)

	function findIndex(array, func) {
	        var length = array.length;
	        var value;
	        for (var i = 0; i < length; i++) {
	            value = array[i];
	            if (func.call(null, value, i, array)) {
	                return i;
	            }
	        }
	        return -1;
	    }
	
	    function isBigEnough(element, index, array) {
	        return element >= 10;
	    }
	
	    var index = findIndex([9, 5, 8, 13, 12], isBigEnough); // 3

### Array.prototype.indexOf() Polyfill   

indexOf() 方法返回在数组中可以找到给定元素的第一个索引，如果不存在，则返回-1。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)
	    
	   /**
	     *
	     * @param array 被查找的数组
	     * @param searchElement 要查找的元素
	     * @param fromIndex 开始查找的位置,-2 表示从倒数第二个元素开始查找
	     * @returns {*}
	     */
	    function indexOf(array, searchElement, fromIndex) {
	
	        var len = array.length;
	        var n = +fromIndex || 0;
	
	        if (len === 0 || n >= len) {
	            return -1;
	        }
	
	        var key = Math.max(n >= 0 ? n : len - Math.abs(n), 0);
	
	        while (key < len) {
	            if (key in array && array[key] === searchElement) {
	                return key;
	            }
	            key++;
	        }
	        return -1;
	    }
	
	    indexOf([9, 5, 8, 13, 3, 5, 12], 5) // 1
	    indexOf([9, 5, 8, 13, 3, 5, 12], 5, 2) // 5	    
	    
	    
 
	        
	      
### Array.prototype.filter() Polyfill

filter() 方法使用指定的函数测试所有元素，并创建一个包含所有通过测试的元素的新数组。

	function filter(array, fun) {
	        var len = array.length;
	        var res = [];
	        for (var i = 0; i < len; i++) {
	            if (i in array) {
	                var val = array[i];
	
	                if (fun.call(null, val, i, array))
	                    res.push(val);
	            }
	        }
	        return res;
	    }
	
	    function isBigEnough(element, index, array) {
	        return element >= 10;
	    }
	
	    var array = filter([10, 9, 5, 8, 13, 12], isBigEnough); // [10, 13, 12]


### Function.prototype.bind() Polyfill

	  Function.prototype.bind = function (oThis) {
	        var args = Array.prototype.slice.call(arguments, 1),
	            functionToBind = this,
	            fNOP = function () {
	            },
	            fBound = function () {
	                return functionToBind.apply(this instanceof fNOP ? this : oThis || this,
	                    args.concat(Array.prototype.slice.call(arguments)));
	            };
	
	        fNOP.prototype = this.prototype;
	        fBound.prototype = new fNOP();
	
	        return fBound;
	    };
	    
