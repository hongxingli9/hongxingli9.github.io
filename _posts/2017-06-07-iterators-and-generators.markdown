---
layout: post
title:  "迭代器与生成器"
date:   2017-06-07 
categories: ES6
---

## 什么是迭代器
迭代器只是带有特殊接口的对象。所有迭代器对象都带有next()方法并返回一个带有两个属性的结果对象。value和done,前者代表下一个位置的值，后者在没有更多值可供迭代的时候返回true。
## 什么是生成器
生成器是返回迭代器的函数，生成器函数由function和之后的*号标示，同时还能使用yield。

```javascript
function *createIterator() {
  yield 1;
  yield 2;
  yield 3;
}
let iterator = createIterator();
console.log(iterator.next().value);  //1
console.log(iterator.next().value);  //2
console.log(iterator.next().value);  //3
```
> yield关键字只能用在生成器内部，在其他地方甚至在生成器内部的函数中使用都会抛出语法错误。

> 无法使用箭头函数来创建生成器。

## 可迭代类型与for-of
与迭代器紧密相关的是，可迭代类型是指那些包含Symbol.iterator属性的对象。

> All iterators created by generators are also iterables, because generators assign the Symbol.iterator property by default.

### 访问默认迭代器
可以使用Symbol.iterator来访问对象默认迭代器。

```javascript
let values = [1,2,3],
    iterator = values[Symbol.iterator]();
console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```
可以如下使用来确定对象是否可迭代

```javascript
function isIterable(object) {
  return typeof object[Symbol.iterator] === "function";
}
console.log(isIterable([1, 2, 3]));     // true
console.log(isIterable("Hello"));       // true
console.log(isIterable(new Map()));     // true
console.log(isIterable(new Set()));     // true
console.log(isIterable(new WeakMap())); // false
console.log(isIterable(new WeakSet())); // false
```
### 创建可迭代类型
开发者自定义的对象默认是不可迭代类型，但是我们可以创建Symbol.iterator属性并指定一个生成器来使这些对象可迭代。

```javascript
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```

## 内置的迭代器
### 集合迭代器
ES6内置三种类型的集合对象：数组，map，set。它们都有如下的迭代器供浏览数据：

* entries() - 返回一个数据集为集合中键值对的迭代器
* values()  - 返回一个数据集为集合中的值的迭代器
* keys()    - 返回一个数据集为集合中的键的迭代器

#### 集合类型的默认迭代器
每种集合类型都包含一种默认的迭代器以供for-of循环显式或隐式的调用。数组和set默认使用values(),map默认使用entries()。
weak set和weak map没有迭代器，使用弱引用意味着没办法知道切确的知道集合中有多少项，于是迭代它们也是不可能的。

### 字符串迭代器
ES5对字符串启用来方括号来访问字符，但是方括号语法访问的是编码单元而非字符本身，所以当获取双字节字符时会有意想不到的问题。

```javascript
var message = "A ð ®· B";

for (let i=0; i < message.length; i++) {
    console.log(message[i]);
}
/*
A
(blank)
(blank)
(blank)
(blank)
B
*/
```
ES6中字符默认迭代器作用的是字符本身而非编码单元。

```javascript
var message = "A ð ®· B";

for (let c of message) {
    console.log(c);
}
/*
A
(blank)
ð ®·
(blank)
B
*/
```

### NodeList的迭代器
在ES6中添加来默认迭代器后，关于NodeList DOM规范中(其实是HTML规范规定而非ES6)也添加了默认迭代器。

## 迭代器的高级用法

### 向迭代器传入参数

```javascript
function *createInterator() {
  let first = yield 1;
  let second = yield first + 2;
  yield second + 3;
}
let iterator = createInterator();
console.log(iterator.next());  //"{value:1,done:false}"
console.log(iterator.next(4));  //"{value:6,done:false}"
console.log(iterator.next(5));  //"{value:8,done:false}"
console.log(iterator.next());  //"{value:undefined,done:true}"
```
首次调用next()有些特殊，传给它的参数都会被忽略。
第二次调用next()时，传入的参数4被赋值给生成器内部的first变量。

### 在迭代器中抛出错误

```javascript
function *createIterator() {
    let first = yield 1;
    let second;

    try {
        second = yield first + 2;       // yield 4 + 2, then throw
    } catch (ex) {
        second = 6;                     // 有错误发生，给 second 赋另外的值
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next());                   // "{ value: undefined, done: true }"
```
### 包含return语句的生成器
生成器会将return语句的出现判断为所有任务已处理完毕，done属性赋值为 true。

### 生成器代理
在某些情况下，将两个迭代器集中到一起会更实用。生成器可以使用yield和星号*这种特殊形式来代理其他生成器。

```javascript
function *createNumberIterator() {
    yield 1;
    yield 2;
}

function *createColorIterator() {
    yield "red";
    yield "green";
}

function *createCombinedIterator() {
    yield *createNumberIterator();
    yield *createColorIterator();
    yield true;
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "red", done: false }"
console.log(iterator.next());           // "{ value: "green", done: false }"
console.log(iterator.next());           // "{ value: true, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```