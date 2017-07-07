---
layout: post
title:  "笔记:set and map"
date:   2017-06-06 
categories: ES6
---

## ECMAScript 5 中的set和map
ES5中用对象来模拟set和map,大部分情况，set用来验证键的存在，map用来提取数据。

## 使用对象模拟的问题
* 在内部，数字类型的键会被转化为字符串，所以 map["5"] 和 map[5] 引用了相同的属性.
* 使用对象作为键的时候也会出现麻烦,在这里，map[key2] 和 map[key1] 引用了相同的值。对象中的 key1 和 key2 被转化为字符串是因为对象的属性只能为该类型。既然对象的字符串表达形式是 "[object Object]"，那么 key1 和 key2 也不例外。默认的字符串转化使得对象很难被当作键来使用。

  ```javascript
  let map = Object.create(null),
    key1 = {},
    key2 = {};

    map[key1] = "foo";

    console.log(map[key2]);     // "foo"  

  ```
* 当键值可以被视为真值时，就会存在歧义，到底是检查值是否存在还是它的值是否为假
  ```javascript
  let map = Object.create(null);

  map.count = 1;

  // 检查 "count" 是否存在或该值是否为假？
  if (map.count) {
    // ...
  }
  ```
## ES6 中的set
ES6中的set类型是一个包含无重复元素的有序列表，允许对内部某元素是否存在进行快速的检查，使得元素的追踪操作效率更高。
### 建立set并添加项
```javascript
let set = new Set();
set.add(5);
set.add("5");
console.log(set.size);  //2
```
set在比较值的时候不会进行强制类型转换。
可以向set添加多个对象，它们的值会被认作是不想等的：
```javascript
let set = new Set(),
    key1 = {},
    key2 = {};
set.add(key1);
set.add(key2);
console.log(set.size);  //2
```
如果add()方法对同一个参数调用多次，那么首次之后的调用将会被忽略。
可以使用数组来初始化一个set,而且set构造函数会确保使用数组的唯一存在的元素：
```javascript
let set = new Set([1,2,3,4,5,5,5,5,5]);
console.log(set.size);  //5
```
可以使用has()检查某个键是否在 set中。
### 移除项
使用delete()移除单个元素，clear()清空整个set。
### set中的forEach()方法
forEach()方法接收三个参数的回调函数
1. 下一个位置的值
2. 和第一个参数相同的值
3. 操作的set本身
### 将set转化为数组
```javascript
let set = new Set([1,2,3,3,3,3,4,5]),
    array = [...set];
console.log(array);  //[1,2,3,4,5];
```
### weak set
set 类型根据它存储对象的方式，也被称为strong set。只要对set实例的引用存在，那么存储的对象在垃圾回收以释放内存的时候就无法被销毁。

  ```javascript
  let set = new Set(),
      key = {};

  set.add(key);
  console.log(set.size);      // 1

  // 销毁引用
  key = null;

  console.log(set.size);      // 1

  // 重新获得了引用
  key = [...set][0];
  ```
### 创建weak set
```javascript
let set = new WeakSet(),
    key = {};
set.add(key);
console.log(set.has(key));  //true
key = null;
console.log(set.has(key));  //false
```
weak set 和 set的一些差异
1. 在weak set中调用add(),has(),delete()传入的是一个非对象参数，会抛出一个错误
2. weak set不是可迭代的，所以不能使用for-of循环
3. weak set无法暴露自己的迭代器(例如keys()或values()方法)，所以没有任何编程手段来确定weak set 中的内容
4. weak set没有forEach()方法
5. weak set 没用size属性

## ES6 中的map
ES6 中的map包含一组有序的键值对，其中键和值可以是任何类型。键的比较结果由Object.is()决定，所以可以用5和"5"来作键，因为它们不是相同的类型。
```javascript
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2016);

console.log(map.get("title"));  //"Understanding ES6"
console.log(map.get("year"));   //2016;
```
如果键不存在，get()返回undefined.
可以用对象用作键。
### map 方法
* has(key) - 判断给定的key是否在map中存在
* delete(key) - 移除map中的key和相应的值
* clear() - 清空map中的键值对
map中size属性指明它包含多少键值对。
### 初始化map
可以将数据存入数组并传给map构造函数来初始化。数组中的每一项必须也是数组，后者包含的的两项中前者为键，后者为值。因此整个map被带有两个项的数组填充
```javascript
let map = new Map([["title","Understanding ES6"], ["year",2016]]);
console.log(map.has("title"));  //true
console.log(map.get("title"));  //"Understanding ES6"
console.log(map.has("year"));   //true
console.log(map.get("year"));   //16
console.log(map.size);          //2
```
### map中的forEach()方法
它同样有三个参数的回调函数
1. map中下一个位置的值
2. 该值对应的键
3. map本身
### weak map
weak map的所有键必须是对象，否则会抛出错误。

> 弱引用指的是键的弱引用，而不是值的弱引用。

#### 使用weak map
#### 初始化weak map
键必须是非null的引用类型。

#### weak map的方法
只有两种方法和键值对交互，has()和delete().没有clear()方法，因为需要对键进行枚举。

#### 私有对象属性
One practical use of weak maps is to store data that is private to object instances.
```javascript
let Person = (function() {

    let privateData = new WeakMap();

    function Person(name) {
        privateData.set(this, { name: name });
    }

    Person.prototype.getName = function() {
        return privateData.get(this).name;
    };

    return Person;
}());
```
#### weak map的实践与限制
如果只想用对象作为键，最好的选择就是weak map，它能优化内存的占用并通过在对象被销毁之后删除额外的信息防止内存泄漏。
weak map稍微减少来自身内容的可见度，不能使用forEach(),size,clear()集中管理集合中的项。
如果使用非对象作为键，那map就是唯一选择。