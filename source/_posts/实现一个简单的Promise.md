---
title: 实现一个简单的 Promise
date: 2017-06-10 08:55:45
tags:
---

随着 JavaScript 异步编程的发展，出现诸如 Promise，Async 等一些异步编程解决方案。

所谓 Promise，其实是对异步操作的封装，它保存着某个未来才会结束的事件的结果。
根据 [Promises/A+]( https://promisesaplus.com/) 规范，Promise 对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称Fulfilled）和 Rejected（已失败）。

只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

关于 Promise 的使用可以参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。本文的主要目的是实现一套简单的 Promise API 以了解它的背后原理。本文 Promise 实现 [源代码](https://github.com/bigggge/P.js)。

## 基础实现

下面是一个简单的 Promise 使用例子

```javascript
getUserId().then(function (id) {
  console.log(id);
});

function getUserId () {
  return new Promise(function (resolve) {
    // 因为执行fn时then还没有执行，所以callback为null。
    // 通过 setTimeout 把 callback 的执行时间推迟到下一事件循环
    setTimeout(function () {
      resolve(1);
    }, 0);
  });
}
```

我们该怎么实现它呢？可以看到我们向 Promise 传入一个函数 fn 并执行了它，传入的函数被给予一个名为 resolve 的参数，resolve 参数用于在合适的时机触发，比如 HTTP 请求返回了结果。

同时，resolve 函数还接收一个参数，即异步操作返回的结果，方便 then 方法中传入的回调函数使用。下面给出了 Promise 最基础的实现：

```javascript
function Promise (fn) {
  var callback = null;
  this.then = function (cb) {
    callback = cb;
  };
  function resolve (value) {
    // 触发回调函数
    callback(value);
  }

  fn(resolve);
}
```

## 多个回调函数

有时候 Promise 需要多个回调函数，就像下面这样：

```javascript
getUserId()
  .then(function (id) {
    console.log('first!');
    console.log(id);
  })
  .then(function (id) {
    console.log('again!');
    console.log(id);
  });

function getUserId () {
  return new Promise(function (resolve) {
    resolve(1);
  });
}
```

很简单，只需要用一个数组保存回调函数就可以了，同时为了能够链式调用，我们还让 then 方法返回了this，
规范明确要求回调需要通过异步方式执行，所以将 setTimeout 从外部移入了 Promise 内部，代码如下：

```javascript
function Promise (fn) {
  var callbacks = [];
  this.then = function (cb) {
    callbacks.push(cb);
    return this;
  };
  function resolve (value) {
    setTimeout(function () {
      callbacks.forEach(function (callback) {
        callback(value);
      });
    }, 0);
  }

  fn(resolve);
}
```

## 引入状态

考虑以下情况

```javascript
function getUserId () {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(1);
    }, 1000);
  });
}

var promise = getUserId();

setTimeout(function () {
  promise.then(function (id) {
    console.log(id);
  });
}, 2000);
```

不难发现，如果 Promise 异步操作已经成功，此时 callbacks 数组还没有内容，之后再通过 then 方法添加的回调函数也不会执行了，这显然不符合我们的预期。根据规范，我们需要用 State 来解决这个问题。

```javascript
function Promise (fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  this.then = function (cb) {
    if (state === 'pending') {
      callbacks.push(cb);
      return this;
    }
    cb(value);
    return this;
  };

  function resolve (newValue) {
    value = newValue;
    state = 'fulfilled';
    setTimeout(function () {
      callbacks.forEach(function (callback) {
        callback(value);
      });
    }, 0);
  }

  fn(resolve);
}
```

resolve 执行时，会将状态设置为 fulfilled，在此之后调用 then 添加的新回调，都会立即执行，即 state 不为 pending 状态时回调会立即执行

## 串行 Promise

我们经常会有下面这样的需求，获取用户 id 后，再根据用户 id 获取用户手机号等其他信息。串行 Promise 是指在当前 Promise 达到 fulfilled 状态后，即开始进行下一个 Promise。

```javascript
function getUserId () {
  return new Promise(function (resolve) {
      resolve(9876);
  });
}

function getUserMobileById (id) {
  return new Promise(function (resolve) {
    console.log('start to get user mobile by id:', id);
      resolve(id + ':' + 13810001000);
  });
}

getUserId()
  .then(getUserMobileById)
  .then(function (mobile) {
    console.log('do sth with', mobile);
  });
```

这个功能的难点是我们应该怎么衔接当前 Promise 与下一个 Promise 呢? 于是我们对 then 方法进行改造：

```javascript
this.then = function (onResolved) {
  return new Promise(function (resolve) {
    if (state === 'pending') {
      callbacks.push([onResolved, resolve]);
      return;
    }
    var ret = onResolved(value);
    resolve(ret);
  });
};
```

then 方法中创建了一个新的 Promise 并作为返回值，这类 Promise 可以被称做 bridge promise。我们将已完成状态的回调函数 onResolved 和 新 Promise 的 resolve 参数传入数组，由上文可知，resolve 函数用于在合适的时机触发。
```javascript
var ret = onResolved(value);
resolve(ret);
```
这段代码也非常重要，这意味着当前 Promise 异步操作成功后执行 onResolved 方法，然后将其返回值作为参数传给新Promise 的 resolve 方法并执行，而也这标志着新 Promise 异步操作成功，Promise 的衔接就这样完成了。

我们还需要在 resolve 方法里判断参数是不是 Promise 的实例，是的话就调用实例的 then 方法。

```javascript
function Promise (fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  this.then = function (onResolved) {
    return new Promise(function (resolve) {
      if (state === 'pending') {
        callbacks.push([onResolved, resolve]);
        return;
      }
      var ret = onResolved(value);
      resolve(ret);
    });
  };

  function resolve (newValue) {
    if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
      var then = newValue.then;
      if (typeof then === 'function') {
        then(resolve);
        return;
      }
    }
    state = 'fulfilled';
    value = newValue;
    setTimeout(function () {
      callbacks.forEach(function (handler) {
        var onResolved = handler[0];
        var resolve = handler[1];
        var ret = onResolved(value);
        resolve(ret);
      });
    }, 0);
  }

  fn(resolve);
}
```

## 失败处理

我们稍微扩展下代码就能进行异常处理了，在异步操作失败时，标记其状态为 rejected，并执行注册的失败回调：

```javascript
function getUserId () {
  return new Promise(function (resolve, reject) {
    setTimeout(function () {
      Math.random() > 0.3 ? resolve(9876) : reject('error1');
    });
  });
}

function getUserMobileById (id) {
  return new Promise(function (resolve, reject) {
    console.log('start to get user mobile by id:', id);
    setTimeout(function () {
      Math.random() > 0.3 ? resolve(id + ':' + 13810001000) : reject('error2');
    });
  });
}

getUserId()
  .then(getUserMobileById)
  .then(function (mobile) {
    console.log('do sth with', mobile);
  })
  .catch(function (reason) {
    console.error(reason);
  });

function Promise (fn) {
  var state = 'pending',
    value = null,
    callbacks = [];

  /**
   * then() 方法返回一个  Promise 它最多需要有两个参数：Promise的成功和失败情况的回调函数。
   *
   * @param onResolved 已完成回调函数
   * @param onRejected 已失败回调函数
   * @return {Promise}
   */
  this.then = function (onResolved, onRejected) {
    return new Promise(function (resolve, reject) {
      if (state === 'pending') {
        callbacks.push([onResolved, resolve, onRejected, reject]);
        return;
      }

      if (state === 'fulfilled') {
        if (onResolved) {
          var ret = onResolved(value);
          resolve(ret);
        } else {
          resolve(value);
        }
      } else {
        if (onRejected) {
          onRejected(value);
        } else {
          reject(value);
        }
      }
    });
  };

  /**
   * catch() 方法返回一个Promise，只处理拒绝的情况。
   * 它的行为与调用 then(undefined, onRejected) 相同。
   *
   * @param onRejected
   * @return {Promise}
   */
  this.catch = function (onRejected) {
    return this.then(null, onRejected);
  };

  /**
   * resolve 可以在外部操作成功时调用，也可能会在 Promise 实现的内部被调用
   *
   * @param newValue 操作成功返回的结果，方便 onResolved 回调函数使用
   */
  function resolve (newValue) {
    if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
      var then = newValue.then;
      if (typeof then === 'function') {
        then(resolve, reject);
        return;
      }
    }
    state = 'fulfilled';
    value = newValue;
    setTimeout(function () {
      callbacks.forEach(function (handler) {
        var onResolved = handler[0];
        var resolve = handler[1];
        if (onResolved) {
          var ret = onResolved(value);
          resolve(ret);
        } else {
          resolve(value);
        }
      });
    }, 0);
  }

  /**
   * reject 可以在外部操作失败时调用，也可能会在 Promise 实现的内部被调用
   *
   * @param reason 操作成功返回的结果，方便 onRejected 回调函数使用
   */
  function reject (reason) {
    value = reason;
    state = 'rejected';
    setTimeout(function () {
      callbacks.forEach(function (handler) {
        var onRejected = handler[2];
        var reject = handler[3];
        if (onRejected) {
          onRejected(value);
        } else {
          reject(value);
        }
      });
    }, 0);
  }

  fn(resolve, reject);
}
```

到这里只是实现了一个比较简单的 Promise，还有诸如 `Promise.race `, `Promise.all  ` 等方法有待以后去实现。

## 最佳实践

不要把 Promise 写成嵌套结构

```
// 错误的写法
promise.then(function(value) {
  promise.then(function(value) {
    promise.then(function(value) {

    })
  })
})
```

链式 Promise 要返回一个 Promise，而不只是构造一个 Promise

```
// 错误的写法
Promise.resolve(1).then(function(){
  Promise.resolve(2)
}).then(function(){
  Promise.resolve(3)
})
```


参考资料：

[Promises/A+规范](http://www.ituring.com.cn/article/66566)  
[剖析 Promise 之基础篇](http://tech.meituan.com/promise-insight.html)  
[剖析Promise内部结构，一步一步实现一个完整的、能通过所有Test case的Promise类](https://github.com/xieranmaya/blog/issues/3)