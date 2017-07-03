---
layout: post
title:  "改进的数组功能(未完成)"
date:   2017-06-10 
categories: ES6
---

## 创建数组
### Array.of() 方法

```javascript
let items = Array.of(1, 2);
console.log(items.length);          // 2
console.log(items[0]);              // 1
console.log(items[1]);              // 2

items = Array.of(2);
console.log(items.length);          // 1
console.log(items[0]);              // 2

items = Array.of("2");
console.log(items.length);          // 1
console.log(items[0]);              // "2"
```
###  Array.from()方法
Given either an iterable or an array-like object as the first argument, the Array.from() method returns an array. Here’s a simple example:

```javascript
function doSomething() {
    var args = Array.from(arguments);

    // use args
}
```
#### Mapping Conversion

```javascript
function translate() {
  return Array.from(arguments, (value) => value + 1);
}
let numbers = translate(1,2,3);
console.log(numbers);  //[2,3,4]
```
If the mapping function is on an object, you can also optionally pass a third argument to Array.from() that represents the this value for the mapping function.

#### Use on Iterables

```javascript
let numbers = {
    *[Symbol.iterator]() {
        yield 1;
        yield 2;
        yield 3;
    }
};

let numbers2 = Array.from(numbers, (value) => value + 1);

console.log(numbers2);              // 2,3,4
```
> If an object is both array-like and iterable, then the iterator is used by Array.from() to determine the values to convert.

## New Methods on All Arrays 
### The find() and findIndex() Methods

```javascript
let numbers = [25, 30, 35, 40, 45];

console.log(numbers.find(n => n > 33));         // 35
console.log(numbers.findIndex(n => n > 33));    // 2
```
### The fill() Method
The fill() method fills one or more array elements with a specific value.

```javascript
let numbers = [1, 2, 3, 4];

numbers.fill(1);

console.log(numbers.toString());    // 1,1,1,1
```
If you only want to change some of the elements, rather than all of them, you can optionally include a start index and an exclusive end index, like this:

```javascript
let numbers = [1, 2, 3, 4];

numbers.fill(1, 2);

console.log(numbers.toString());    // 1,2,1,1

numbers.fill(0, 1, 3);

console.log(numbers.toString());    // 1,0,0,1
```
> 起始位置或结束为止为负数时，index加上数组长度后得到最后的位置值。

### The copyWithin() Method

```javascript
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
numbers.copyWithin(2, 0);

console.log(numbers.toString());    // 1,2,1,2
```
```javascript
let numbers = [1, 2, 3, 4];

// paste values into array starting at index 2
// copy values from array starting at index 0
// stop copying values when you hit index 1
numbers.copyWithin(2, 0, 1);

console.log(numbers.toString());    // 1,2,1,4
```

## 类型数组
