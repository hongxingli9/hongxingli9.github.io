---
layout: post
title:  "javascript 隐式类型转换"
date:   2017-07-07
categories: javascript
---

## javascript隐形类型转换步骤

这里说的隐性类型转换，是==引起的转换。

1. 如果存在NaN，一律返回false
2. 再看有没有布尔，有布尔就将布尔转换为数字
3. 接着看有没有字符串, 有三种情况，对方是对象，对象使用toString进行转换；对方是数字，字符串转数字；对方是字符串，直接比较；其他返回false
4. 如果是数字，对方是对象，对象取valueOf进行比较, 其他一律返回false
5. null, undefined不会进行类型转换, 但它们俩相等

### 题目
1. 0 == undefined
2. 1 == true
3. 2 = {valueOf: function() {return 2;}}
4. NaN == NaN
5. 8 == undefined
6. 1 == undefined
7. null == {toString: function() {return 2;}}
8. 0 == null
9. null == 1
10. {toString: function() {return 1}, valueOf: function() {return []}} == 1

