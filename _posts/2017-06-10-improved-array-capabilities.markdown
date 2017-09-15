---
layout: post
title:  "笔记:改进的数组功能"
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

console.log(numbers.toString());    // 1,0,0,4
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

## 类型化数组
类型化数组是有特殊用途的数组，被设计用来处理数值类型数据。类型化数组可以追溯到WebGL--OpenGL ES 2.0的一个接口，设计用于配合网页上的\<canvas\>元素。类型化数组作为接口实现的一部分，为js提供了快速的按位运算能力。

### 数值数据类型
类型化数组允许存储并操作八种不同的数值类型：
* 8位有符号整数(int8)
* 8位无符号整数(uint8)
* 16位有符号整数(int16)
* 16位无符号整数(uint16)
* 32位有符号整数(int32)
* 32位无符号整数(uint32)
* 32位浮点数(float32)
* 64位浮点数(float64)

### 数组缓冲区
数组缓冲区是内存里保存一定数量字节的区域，所有的类型化数组都基于数组缓冲区。

```javascript
let buffer = new ArrayBuffer(10); //allocate 10 bytes
console.log(buffer.byteLength);  //10
```
可以使用slice()方法创建新的，包含已有缓冲区部分内容的缓冲区。

```javascript
let buffer = new ArrayBuffer(10);
let buffer2 = buffer.slice(4, 6);
console.log(buffer2.byteLength);   //2
```

> 数组缓冲区总是保持创建时指定的字节数，可以修改其内部的数据，但是不能修改它的容量。

### 使用视图操作数组缓冲区
数组缓冲区代表一块内存区域，而视图(view)则是你操作这块区域的接口。

```javascript
let buffer = new ArrayBuffer(10),
	view = new DataView(buffer);
```

```javascript
let buffer = new ArrayBuffer(10),
	view = new DataView(buffer, 5, 2);  //cover bytes 5 and 6
```

#### 获取视图信息
* buffer 视图所绑定的数组缓冲区
* byteOffset DataView构造函数的第二个参数，如果提供了的话(默认为0)
* byteLength DataView构造函数的第三个参数,如果提供了的话(默认为buffer's byteLength)

```javascript
let buffer = new ArrayBuffer(10),
	view1 = new DataView(buffer),
	view2 = new DataView(buffer, 5, 2);
console.log(view1.buffer === buffer);	//true;
console.log(view2.buffer === buffer);  	//true;
console.log(view1.byteOffset);	//0
console.log(view2.byteOffset); 	//5
console.log(view1.byteLength);  //10
console.log(view2.byteLength);	//2
```

#### 读取和写入数据
* getInt8(byteOffset, littleEndian)	在byteOffset处读取一个int8值
* setInt8(byteOffset, value, littleEndian) 在byteOffset处写入一个int8值
* getUint8(byteOffset, littleEndian) 在byteOffset处读区一个无符号int8值
* setUint8(byteOffset, value, littleEndian) 在byteOffset处写入一个无符号int8值

> little endian 小端子节序，指的是在存储数据的多个内存字节中，第一个内存字节存储着数据的最低字节数据，最后一个内存字节存储着数据的最高字节数据。

```javascript
let buffer = new ArrayBuffer(2),
	view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt8(0));	//5
console.log(view.getInt8(1));	//-1
```

视图允许使用任意格式对任意位置进行读写，而无须考虑这些数据此前是以什么格式存储的。

```javascript
let buffer = new ArrayBuffer(2),
	view = new DataView(buffer);

view.setInt8(0, 5);
view.setInt8(1, -1);

console.log(view.getInt16(0));	//1535
console.log(view.getInt8(0));	//5
console.log(view.getInt8(1));	//-1

```

new ArrayBuffer(2)  	0000000000000000
view.setInt8(0, 5)		0000010100000000
view.setInt8(1, -1)		0000010111111111

#### 类型化数组即是视图

#### 创建特定类型视图

第一种方式是可以像创建DataView时相同的参数

```javascript
let buffer = new ArrayBuffer(10),
    view1 = new Int8Array(buffer),
    view2 = new Int8Array(buffer, 5, 2);

console.log(view1.buffer === buffer);	// true
console.log(view2.buffer === buffer);	// true
console.log(view1.byteOffset);			// 0
console.log(view2.byteOffset);			// 5

console.log(view1.byteLength);              // 10
console.log(view2.byteLength);              // 2
```
第二种方式是向构造函数单独传递一个数值作为参数，该数值表示数组包含的元素数量。

```javascript
let ints = new Int16Array(2),
    floats = new Float32Array(5);
console.log(ints.byteLength);  // 4 
console.log(ints.length);   // 2
console.log(floats.byteLength);   // 20
console.log(floats.length);    // 5
```

第三种方式是传递单个对象参数，可以是下列四种对象之一：
* 类型化数组
* 可迭代对象
* 数组
* 类数组对象

## 类型化数组与常规数组的相似点

可以使用length属性来获取类型化数组包含的元素数量， 还可以使用数值类型的索引值来直接访问类型化数组的元素。但不像常规数组，类型化数组不能修改length属性的值来改变数组的大小。

### 常见的方法
类型化数组拥有很多和常规数组等效的方法
copyWithin()	entries()	fill()		filter()		find()
findIndex()		forEach()	indexOf()	join()			keys()
lastIndexOf()	map()		reduce()	reduceRight()	reverse()
slice()			some()		sort()		values()

类型化数组会进行额外的类型检查以确保安全，并且返回值会是类型化数组

### 相同的迭代器

entries()		keys()		values()

### of()和from()方法

## 类型化数组和常规数组的区别

类型化数组不是常规数组类型，并不是Array对象派生出来的，使用Array.isArray()去检测会返回false。
常规数组可以通过交互改变数组的大小，但是类型化数组是固定大小的。
类型化数组会对数据类型检查以保证只使用有效的值，0会代替所有无效的值。
字符串对于类型化数组来说是无效的值。

### 缺少的方法

concat() shift() pop() splice() push() unshift()

### 附加的方法

set()
```javascript
let ints = new Int16Array(4);
ints.set([25, 50]);
ints.set([75, 100], 2);
console.log(ints.toString());   // 25,50,75,100
```

subarray()
```javascript
let ints = new Int16Array([25, 50, 75, 100]),
    subints1 = ints.subarray(),
    subints2 = ints.subarray(2),
    subints3 = ints.subarray(1, 3);
console.log(subints1.toString());		// 25,50,75,100
console.log(subints2.toString());		// 75,100
console.log(subints3.toString());		// 50,75
```
