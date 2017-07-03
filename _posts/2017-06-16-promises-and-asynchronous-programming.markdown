---
layout: post
title:  "Promises 与异步编程"
date:   2017-06-16 
categories: ES6
---

## Promise的基础
promise是异步操作结果的占位符。

### promise的生命周期
每个promise的生命周期从一开都会处于一个短暂的挂起(pending)状态，表示异步操作仍未完成。即挂起的promise被认定是未定的(unsettled)。一旦异步操作完成，promise就会被认为是已定(settled)的并处于下面两种状态之一：
1. fulfilled: promise的异步操作已完成。
2. rejected: promise的异步操作未完成，可能是发生了错误或其他原因。
所有的promise都包含then()方法并接受两个参数，第一个参数是fulfilled状态下调用的函数，任何异步操作的有关的额外数据都会传给它。第二个参数是rejected状态下调用的函数，它会被传入任何与操作未完成有关的的数据。

> 以该种方式实现then()方法的对象被称为thenable.所有的promise都是thenable，但不是所有的thenable都是promise。

then()中的两个参数是可选的，所以可以同时对fulfilled和rejected或对其一进行监听。

```javascript
let promise = readFile("example.txt");
promise.then(function(content) {
  //fulfillment
  console.log(content);
}, function(err) {
  //rejection
  console.error(err.message);
});
promise.then(function(content) {
  //fulfillment
  console.log(content);
});
promise.then(null, function(err) {
  //rejection
  console.error(err.message);
})
```
promise也包含一个catch()方法，它的行为等效于只传递rejection处理。

```javascript
promise.catch(function(err) {
  console.error(err.message);
});
// 等效于
promise.then(null, function(err) {
  console.error(err.message);
});
```
A fulfillment or rejection handler will still be executed even if it is added to the job queue after the promise is already settled.

```javascript
let promise = readFile("example.txt");

// 原始的 fulfillment 处理
promise.then(function(contents) {
    console.log(contents);

    // 新添加的处理
    promise.then(function(contents) {
        console.log(contents);
    });
});
```
### 创建未定的promise
promise由Promise的构造函数创建，该函数接收一个参数:包含初始化promise代码的执行函数。执行函数接收两个参数：resolve()和reject()。resolve()在执行函数成功运行后发出信号表示该promise已经可用。而reject()函数代表该执行函数运行失败。

```javascript
let fs = require("fs");
function readFile(filename) {
  return new Promise(function(resolve, reject) {
    fs.readFile(filename, {encoding: "utf8"}, function(err, contents) {
      if (err) {
        reject(err);
        return;
      }
      resolve(contents);
    });
  });
}
let promise = readFile("example.txt");
promise.then(function(content) {
  console.log(content);
}, function(err) {
  console.error(err.message);
});

```
Calling resolve() triggers an asynchronous operation. Functions passed to then() and catch() are executed asynchronously, because these are also added to the job queue. Here’s an example:

```javascript
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

promise.then(function() {
    console.log("Resolved.");
});

console.log("Hi!");

//Promise
//Hi!
//Resolved
```
虽然调用then()在console.log("Hi!")之前，实际上它并不会立即执行(和执行函数不同)。这是因为fulfillment和rejection处理总是会在执行函数运行完毕后被添加到任务列表的末尾。

### 创建已定promise
#### 使用Promise.resolve()
Promise.resolve()接收单个参数并返回一个fulfilled状态的promise。

```javascript
let promise = Promise.resolve(42);
promise.then(function(value) {
  console.log(value);  //42
});
```
如果rejection处理被添加到这个promise,那么它永远也不会被调用。
Promise.reject()方法来创建rejected的promise。

```javascript
let promise = Promise.reject(42);
promise.catch(function(value) {
  console.log(value);  //42
});
```
> If you pass a promise to either the Promise.resolve() or Promise.reject() method, the promise is returned without modification.

#### 非promise的thenable对象
Both Promise.resolve() and Promise.reject() also accept non-promise then- ables as arguments. When passed a non-promise thenable, these methods create a new promise that is called after the then() function.

```javascript
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
    console.log(value);     // 42
});
```
In this example, Promise.resolve() calls thenable.then() so a promise state can be determined. The promise state for thenable is fulfilled because resolve(42) is called inside the then() method. A new promise called p1 is created in the fulfilled state with the value passed from thenable (that is, 42), and the fulfillment handler for p1 receives 42 as the value.

#### 执行错误
如果在执行函数里面抛出错误，promise的rejection处理会被调用

```javascript
let promise = new Promise(function(resolve, reject) {
  throw new Error("Explosion!");
});
promise.catch(function(err) {
  console.error(err.message);  //"Explosion!"
});
```
在每个执行函数里面都有个隐式的try-catch捕获错误并传给rejection处理。

## promise的全局rejection处理

### Node.js中的rejection处理
Node.js emits two events on the process object that are related to promise rejection handling:
* unhandledRejection Emitted when a promise is rejected and no rejec- tion handler is called within one turn of the event loop
* rejectionHandled Emitted when a promise is rejected and a rejection handler is called after one turn of the event loop
The unhandledRejection event handler is passed the rejection reason (frequently an error object) and the promise that was rejected as arguments.

```javascript
et rejected;

process.on("unhandledRejection", function(reason, promise) {
    console.log(reason.message);            // "Explosion!"
    console.log(rejected === promise);      // true
});

rejected = Promise.reject(new Error("Explosion!"));
```
The rejectionHandled event handler has only one argument, which is the promise that was rejected. For example:

```javascript
let rejected;

process.on("rejectionHandled", function(promise) {
    console.log(rejected === promise);              // true
});

rejected = Promise.reject(new Error("Explosion!"));

// 等待 rejection 处理的添加
setTimeout(function() {
    rejected.catch(function(value) {
        console.log(value.message);     // "Explosion!"
    });
}, 1000);
```
### 浏览器中的rejection处理

浏览器中同样设置了两个事件以便来查找未处理的rejection。这些事件由window对象处罚并等效于Node.js的实现。
* unhandledrejection
* rejectionhandled
Node.js的实现中，事件处理函数的参数是分别(reason,promise)传入的，而浏览器的事件处理函数参数接收包含下列属性的的event对象：
* type: 事件的名称
* promise: 处于rejection状态的promise
* reason: promise的rejection值
浏览器实现的差异是两种事件rejection值(reason)都可以使用。

```javascript
let rejected;
window.onunhandledrejection = function(event) {
  console.log(event.type);  //"unhandledrejection"
  console.log(event.reason.message);  //"Explosion!"
  console.log(event.promise === rejected)  //true
};
window.onrejectionhandled = function(event) {
  //...
};
rejected = Promise.reject(new Error("Explosion!"));
```
## promise链
每一次调用then()或catch()都会返回另一个promise。它只会在之前的promise转化为fulfilled或rejected状态的那一刻才会被处理。

```javascript
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});
p1.then(function(value) {
  console.log(value);
}).then(function() {
  console.log("Finished");
});

//42
//"Finished"
```
### 捕捉错误
promise链允许你捕获上一个promise的fufillment或rejection处理中的错误。

```javascript
let p1 = new Promise(funciton(resolve, reject) {
  resolve(42);
});
p1.then(function(value) {
  throw new Error("Boom!");
}).catch(function(error) {
  console.log(error.message);  //"Boom!"
});
```
上一个promise如果在rejection处理中抛出错误，同样能被接收：

```javascript
let p1 = new Promise(funciton(resolve, reject) {
  throw new Error("Explosion!");
});
p1.catch(function(error) {
  console.log(error.message);  //"Explosion!"
  throw new Error("Boom!");
}).catch(function(error) {
  console.log(error.message);  //"Boom!"
});
```
> 为了确保能正确处理可能发生的错误，你总是需要在promise链的末尾添加rejection处理。

### promise链中的返回值
promise链的另一个重要特征是链中的promise能够向下一个promise传递数据。

```javascript
let p1 = new Promise(function(resolve, reject) {
  resolve(42);
});
p1.then(function(value) {
  console.log(value);  //42
  return value + 1;
}).then(function(value) {
  console.log(value);  //43
});
```
### promise链中的promise返回

```javascript
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    // 首个 fulfillment 处理
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // 第二个 fulfillment 处理
    console.log(value);     // 43
});
```
## 响应多个promise
如果要观察多个promise的进度来决定下一步的操作，ES6提供了两个方法负责此事:Promise.all()和Promise.race()
### Promise.all()方法
Promise.all()方法接收单个包含promise的可迭代对象参数(如数组)，并在该对象所有promise全部处理完毕之后返回一个已处理的promise。

```javascript
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.then(function(value) {
    console.log(Array.isArray(value));  // true
    console.log(value[0]);              // 42
    console.log(value[1]);              // 43
    console.log(value[2]);              // 44
});
```
如果其中一个promise转变为rejection状态，那么会立即返回一个rejected状态的promise而不用等其他promise完成执行。

```javascript
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.catch(function(value) {
    console.log(Array.isArray(value))   // false
    console.log(value);                 // 43
});
```
### Promise.race()方法
在单个promise执行完毕的那一刻返回一个promise,只要有promise转化为fulfilled状态，那么Promise.race()就会返回它。

```javascript
let p1 = Promise.resolve(42);

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.then(function(value) {
    console.log(value);     // 42
});
```
## promise继承

```javascript
class MyPromise extends Promise {

    // 使用默认的构造函数

    success(resolve, reject) {
        return this.then(resolve, reject);
    }

    failure(reject) {
        return this.catch(reject);
    }

}

let promise = new MyPromise(function(resolve, reject) {
    resolve(42);
});

promise.success(function(value) {
    console.log(value);             // 42
}).failure(function(value) {
    console.log(value);
});

```
MyPromise.resolve()和MyPromise.reject()会返回MyPromise的实例而无视传给他们的参数，因为这些方法使用了Symbol.species属性来决定返回promise的类型。

```javascript
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = MyPromise.resolve(p1);
p2.success(function(value) {
    console.log(value);         // 42
});

console.log(p2 instanceof MyPromise);   // true
```
### 异步运行器

```javascript
let fs = require("fs");

function run(taskDef) {

    // 创建迭代器
    let task = taskDef();

    // 开始任务
    let result = task.next();

    // 使用函数递归进行迭代
    (function step() {

        // 如果还有工作要做
        if (!result.done) {

            // 使用 resolve() 来简化 promise 的处理
            let promise = Promise.resolve(result.value);
            promise.then(function(value) {
                result = task.next(value);
                step();
            }).catch(function(error) {
                result = task.throw(error);
                step();
            });
        }
    }());
}

// 定义任务运行器需要的函数

function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, contents) {
            if (err) {
                reject(err);
            } else {
                resolve(contents);
            }
        });
    });
}

// 运行一个任务

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});

```