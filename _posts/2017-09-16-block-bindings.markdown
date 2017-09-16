---
layout: post
title:  "笔记:块级绑定"
date:   2017-09-16
categories: ES6
---

## var声明和变量提升

使用关键字var声明变量，无论其实际的声明位置在哪，都会视为声明在函数的顶部(如果声明不在任何函数内，则视为全局作用域的顶部)。这就是所谓的变量提升。

```javascript
function getValue(condition) {
    if (condition) {
        var value = "blue";
		// other code
        return value;
    } else {
            // value exists here with a value of undefined
            return null;
        }
        // value exists here with a value of undefined
    }

```
## 块级声明

块级作用域在以下情况被创建：
* 在一个函数内部
* 在一个代码块里(花括号包裹着)内部

### let声明

可以使用let代替var声明变量，但是会把变量的作用域限制在代码块中。
let声明不会被提升到当前代码块的顶部。

```javascript

function getValue(condition) {
    if (condition) {
        let value = "blue";
		// other code
        return value;
    } else {
        // value doesn't exist here
        return null;
    }
    // value doesn't exist here
}

```

### 禁止重复声明

在相同的作用域，用let重复声明定义一个已经被定义过的标示符，会抛出错误。
如果在包含的作用域里用let声明一个同名的新变量，就不会抛出错误。

```javascript
var count = 30;
if (condition) {
    // doesn't throw an error
    let count = 40;
	// more code 
}
```

### const声明

使用const声明的变量会被视为常量，意味着一旦被赋值就不能再改变。
所有的const变量必须在声明的时候赋值。

#### 对比常量声明和let声明

* 都是块级声明
* 在声明的语句块外面无法再访问到
* 声明不会被提升
* 声明一个已经在相同作用域里定义过的变量会抛出错误
* 无论是严格模式还是非严格模式，尝试赋值给一个已经定义过的常量会抛出错误
* 如果常量是一个对象，对象所包含的值是可以改变的

#### 使用const声明对象

const会阻止对于变量绑定与变量自身值的修改，这意味着不会阻止变量成员的修改。

```javascript
const person = {
    name: "Nicholas"
};
// works
person.name = "Greg";
// throws an error
person = {
    name: "Greg"
};
```

### 暂存死区

在声明const或let变量前都不可以访问该变量。
尝试访问暂存死区的变量会抛出运行时错误。

## 循环中的块级绑定

### 循环内的常量声明

在for-in或for-of循环中，const声明的变量跟let声明的变量表现相似。

const能在for-in或for-of循环中工作，是因为循环为每次迭代创建一个新的变量绑定，而不是修改已绑定的值。

在for循环中使用const声明会导致错误

## 全局块绑定

如果在全局作用域用const或let声明变量，只会在作用域创建新的绑定，但不会被当成属性添加到全局对象上。

