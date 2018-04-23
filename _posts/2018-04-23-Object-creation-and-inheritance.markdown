---
layout: post
title:  "笔记:对象创建和继承"
date:   2018-04-23 
categories: javascript
---

## 对象创建
1. 工厂模式，解决了创建多个相似对象的问题，没有解决对象识别的问题。
2. 构造函数模式，问题：每个方法都要在每个实例重新创建一次
3. 原型模式，问题：所有属性被所有实例共享
4. 组合使用构造函数和原型模式
5. 动态原型模式
6. 寄生构造函数模式，除了使用new操作符并把使用的包装函数叫做构造函数外，跟工厂模式一模一样。不能依赖instanceof操作符确定对象类型
7. 稳妥构造函数模式，没有公共属性，而且其方法也不引用this的对象。与寄生构造函数模式类似，但不引用this,不使用new操作符调用构造函数。


## 继承
1.  原型链

 	原型链问题：
 	1） 包含引用类型值的原型属性会被所有实例共享。
	2） 在创建子类型的实例时，不能向超类的构造函数传递参数。

2. 借用构造函数(伪造对象或经典继承)

	```javascript
	function SubType() {
		SuperType.call(this);
	}
	```

3. 组合继承(伪经典继承)

    原型链和借用构造函数组合在一起的技术。

4. 原型式继承

	```javascript
    function object(o) {
		function F() {};
		F.prototype = o;
		return new F();
	}
	```

5. 寄生式继承

	```javascript
	function createAnother(original) {
		var clone = object(original);
		clone.sayHi = function() {
			alert("Hi");
		};
		retrun clone;
	}
	```
6. 寄生组合式继承

  	借用构造函数来继承属性，通过原型链的混成形式来继承方法。其背后的思想是：不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非是超类型原型的一个副本。
  	
   ```javascript
   function inheritPrototype(subType, superType) {
		var prototype = object(superType.prototype);
		prototype.constructor = subType;
		subType.prototype = prototype;
	}
   ```