---
layout: post
title:  "笔记:ES6 类"
date:   2017-06-09 
categories: ES6
---

## 类声明

```javascript
//class 关键字和紧随其后的命名
class PersonClass {
  //等效于PersonClass的构造函数
  constructor(name) {
    this.name = name;
  }
  //看起来和对象字面量的简写方式类似，不过它们之间不用逗号分隔
  //等效于PersonClass.prototype.sayName
  sayName() {
    console.log(this.name);
  }
}
let person = new PersonClass("Nicolas");
person.sayName();  //"Nicolas"

console.log(person instanceof PersonClass);  //true
console.log(person instanceof Object);       //true
console.log(typeof PersonClass);             //"function"
console.log(typeof PersonClass.prototype.sayName);  //"function"
```
有意思的是，类的声明只是上面例子中自定义类型的语法糖。PersonClass声明实践上创建了一个行为和constructor方法相同的构造函数。

### 为何使用类声明
1. 类声明和函数定义不同，不会被提升。类似与let(TDZ)
2. 类声明的代码自动运行在严格模式下，而且没有办法手动切换回非严格模式
3. 所有的方法都是不可枚举的，这和自定义类型相比是个显著的差异
4. 所有的方法都不能用new来调用，因为它们内部没有[[constrctor]]方法
5. 不使用new调用构造函数会抛出错误
6. 试图在方法内部重写类名的行为会抛出错误

> 在类内部，类名被视为由const声明的常量，这意味着只能在类外部才能对类名进行重写。

### 一个基本的类表达式

```javascript
let PersonClass = class {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};
```
### 具名类表达式

```javascript
let PersonClass = class PersonClass2 {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};
```
类表达式被命名为PersonClass2，该标识符只存在与定义的类内部。在类外部使用typeof PersonClass2为"undefined"是因为不存在任何PersonClass2的绑定。

## 作为一等公民的类

类可以作为函数的参数
类表达式的另一个有趣的使用方式是通过立即调用类构造函数来创建单例。

```javascript
let person = new class {
  constructor(name) {
    this.name = name;
  }
  sayName() {
    console.log(this.name);
  }
}("Nicolas");
```

## 访问器属性

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor);  //true
console.log("set" in descriptor);   //true
console.log(descriptor.enumerable);  //false
```
## 动态计算的成员命名

```javascript
let methodName = "sayName";

class PersonClass {
  constructor(name) {
    this.name = name;
  }
  [methodName]() {
    console.log(this.name);
  }
}
let me = new PersonClass("Nicolas");
me.sayName();
```

## 静态成员

```javascript
class PersonClass {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }

    // 等效于 PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}

let person = PersonClass.create("Nicholas");
```
可以将static关键字添加到除constructor以外的任何方法或访问器属性。

> 静态成员变量不能被实例访问，可以用类本身来使用它们。

## 以派生类为继承方式

```javascript
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle {
    constructor(length) {

        // 等效于 Rectangle.call(this, length, length)
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```
类如果继承了其他类，那么它就是派生类。如果在派生类定义了constructor则必须使用super()，否则会发生错误。
如果选择不使用constructor，那么会自动调用super()和传入构造函数的参数以创建类的实例。

> 只能在派生类中使用super(),否则(没有使用extends的类或函数)会抛出一个错误。
> 必须在构造函数起始位置调用super(),因为它会初始化this,任何在super()之前访问this的行为都会造成错误。
> 在类构造函数中，唯一能避免调用super()的办法是返回一个对象。

### 隐藏类方法
派生类中的方法总是会屏蔽基类中同名的方法。

### 静态成员继承

```javascript
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }

    static create(length, width) {
        return new Rectangle(length, width);
    }
}

class Square extends Rectangle {
    constructor(length) {

        // 等效于 Rectangle.call(this, length, length)
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle);     // true
console.log(rect.getArea());                // 12
console.log(rect instanceof Square);        // false
```
Square.create()调用等同于Rectangle.create()。

### 继承表达式的派生类

ES6派生类可以继承一个表达式，只要该表达式的计算结果是包含[[constructor]]的函数和一个原型，那么就可以使用extends来继承。

```javascript
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true
```

### 内置对象的继承
自 JavaScript 数组存在的那天起，开发者就想通过使用继承的方式来定义特殊的数组类型。在 ECMAScript 5 和更早的版本中，这是不可能做到的。使用传统的继承方式无法获得想要的功能。

```javascript
// 内置数组的行为
var colors = [];
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined

// 尝试使用 ES5 的继承方式

function MyArray() {
    Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
});

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 0

colors.length = 0;
console.log(colors[0]);             // "red"
```
ES6 类的目标之一就是允许继承内置对象。为了实现这一需求，类的继承模型和ES5及更早版本的传统继承模型相比有些细微的不同。

* 在ES5传统继承中，this首先由派生的类型创建，之后才会调用基类的构造函数，这意味着this一开始就是MyArray的实例，Array的属性负责装饰它
* 在ES6的继承中，this首先由基类创建并在之后由派生的构造函数进行修改。于是this一开始就具有内置对象的全部功能并在之后能接收其他功能扩展。

```javascript
class MyArray extends Array {
    // empty
}

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined
```

### Symbol.species属性
关于继承内置对象的一个有趣现象是，任何返回内置对象实例的方法会自动返回派生类的实例。

```javascript
class MyArray extends Array {
    // empty
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);  //slice()方法会返回MyArray的实例
                                   //发生这样的改变归咎于Symbol.species属性在幕后操作
                                   //Symbol.species被用来定义一个返回函数的静态访问器属性
                                   //该属性返回的是构造函数并可随时在实例方法中创建该类的实例
console.log(items instanceof MyArray);      // true
console.log(subitems instanceof MyArray);   // true
```



