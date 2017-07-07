---
layout: post
title:  "笔记:Symbols 与 Symbols属性"
date:   2017-06-05 
categories: ES6
---

Symbol是ECMAScript 6 新引入的基本类型，对象可以使用Symbol来创建私有成员。

## 创建Symbol
```javascript
let firstName = Symbol();
let person = {};

person[firstName] = "Nicolas";
console.log(person[firstName]);
```
>由于Symbols是基本类型，调用 new Symbol()会抛出错误

Symbol 函数也会接收一个参数作为自己的描述
```javascript
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicolas";
console.log("first name" in person);  //false
console.log(person[firstName]);   //"Nicolas"
console.log(firstName);   //"Symbol(first name)"
```

> 可以使用typeof操作符检查变量是否包含symbol
```javascript
let symbol = Symbol("test symbol");
console.log(typeof symbol);  //"symbol"
```

## 共享Symbol
使用Symbol.for()方法生成可以共享的Symbol,调用该方法首次会先检查是否存在某key的symbol, 有则返回，无则生成该key的symbol.

```javascript
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);  //"12345"
console.log(uid);  //"symbol
```

## Symbol 类型转换
Symbols cannot be coerced into strings or numbers so that they cannot accidentally be used as properties that would otherwise be excepted to behave as symbols.

```javascript
let uid = Symbol.for("uid"),
    descript = uid + "";  
    //error. concatenate the symbol directly with a string, an error will be thrown;
    //similarly, you cannot coerce a symbol to a number.
```

## 提取Symbol属性
The Object.keys() and Object.getOwnPropertyNames() methods can retrieve all property names in an object. The former method returns all enumerable property names, and the latter returns all properties regardless of enumerability. 
the Object.getOwnPropertySymbols() method was added in ECMAScript 6 to allow you to retrieve property symbols from an object.
The return value of Object.getOwnPropertySymbols() is an array of own property symbols.

## 公开Well-Known-Symbols的内部操作
ECMAScript 6 has predefined symbols called well-known symbols that represent common behaviors in JavaScript that were previously considered internal-only operations. 
* Symbol.hasInstance - A method used by instanceof to determine an object’s inheritance.
* Symbol.isConcatSpreadable - A Boolean value indicating that Array.prototype.concat() should flatten the collection’s elements if the collection is passed as a parameter to Array.prototype.concat().
* Symbol.iterator - A method that returns an iterator. (Iterators are covered in Chapter 7.)
* Symbol.match - A method used by String.prototype.match() to compare strings.
* Symbol.replace - A method used by String.prototype.replace() to replace substrings.
* Symbol.search - A method used by String.prototype.search() to locate substrings.
* Symbol.species - The constructor for making derived objects. (Derived objects are covered in Chapter 8.)
* Symbol.split - A method used by String.prototype.split() to split up strings.
* Symbol.toPrimitive - A method that returns a primitive value representation of an object.
* Symbol.toStringTag - A string used by Object.prototype.toString() to create an object description.
* Symbol.unscopables - An object whose properties are the names of object properties that should not be included in a with statement.

### the Symbol.hasInstance Property
ECMAScript 6 essentially redefined the instanceof operator as shorthand syntax for this method call. And now that there’s a method call involved, you can actually change how instanceof works.

```javascript
function MyObject() {
  //....
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
  value: function(v){
    return false;
  }
});

let obj = new MyObject();
console.log(obj instanceof MyObject);  //false
```

### The Symbol.isConcatSpreadable Symbol
```javascript
let collection = {
  0: "Hello",
  1: "World",
  length: 2,
  [Symbol.isConcatSpreadable]: true
};
let message = ["Hi"].concat(collection);
console.log(message.length);  //3;
console.log(message);  //["Hi","Hello","World"];
```
### The Symbol.match, Symbol.replace, Symbol.search, and Symbol.split Symbols
```javascript
//effectively equivalent to /^.{10}$/
let hasLengthOf10 = {
  [Symbol.match]: function(value) {
    return value.length === 10 ? [value.substring(0,10)] : null;
  },
  [Symbol.search]: function(value) {
    return value.length === 10 ? 0 : -1;
  },
  [Symbol.replace]: function(value, replacement) {
    return value.length === 10 ? replacement + value.substring(10) : value;
  },
  [Symbol.split]: function(value) {
    return value.length === 10 ? ["",""] : [value];
  }
};
let message1 = "Hello World",  // 11 characters
    message2 = "Hello John";   // 10 characters

let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);  //null
console.log(match2);  //["Hello John"]

let replace1 = message1.replace(hasLengthOf10),
    replace2 = message2.replace(hasLengthOf10);

console.log(replace1);  //"Hello World"
console.log(replace2);  //"undefined"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);  //-1
console.log(search2);  //0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);  //["Hello World"]
console.log(split2);  //["",""]
```

### The Symbol.toPrimitive Symbol

> Default mode is only used for the == operator, the + operator, and when passing a single argument to the Date constructor. Most operations use string or number mode.

```javascript
function Temperature(degree) {
  this.degree = degree;
}
Temperature.prototype[Symbol.toPrimitive] = function(hint) {
  switch (hint) {
    case "string":
      return this.degree + "\u00b0"; // degrees symbol
    case "number":
      return this.degree;
    case "default":
      return this.degree +  "degree";
  }
};

let freezing = new Temperature(32);

console.log(freezing + "!");  //"32 degree"
console.log(freezing / 2);    //16;
console.log(String(freezing));  //"32째"
```
### The Symbol.toStringTag Symbol
```javascript
function supportNativeJSON() {
  return typeof JSON !== "undefined" && Object.prototype.toString.call(JSON) === "[object JSON]";
}
```
A non-native JSON object would return [object Object] while the native version returned [object JSON] instead.

the Symbol.toStringTag symbol represents a property on each object that defines what value should be produced when Object.prototype.toString.call() is called on it.
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

let me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```
