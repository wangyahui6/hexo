---
title: javascript类型检测
date: 2017-09-08 17:43:17
categories: web前端
description: javascript中一共有5中原始类型，Number、String、Boolean、null、undefined；除了原始类型外都是引用类型，统称为对象。引用类型包括Object、Array、Date、RegExp、Error。js判断数据类型有很多方法，typeof、instanceof、Object.prototype.toString.call()等，本文将总结最可靠的判断类型的方法。
tags: javascript
---
### 原始类型的检测

对于原始类型最简单可靠的检测方法就是`typeof`运算符了。
- 对于字符串，typof返回'string'。
- 对于数字，typof返回'number'。
- 对于布尔值，typof返回'boolean'。
- 对于undefined，typof返回'undefined'。

typeof还有一个独特之处，对于未声明的变量不会报错。未声明的变量和值为`undefined`的变量都将返回'undefined';

但是对于`null`,typof返回'object'.对于null，只用===和null比较就可以了。

### 引用类型的检测

typeof对除了函数外的所有的引用类型都返回'object'，所以引用类型不能使用typeof来判断了。
```
    console.log(typeof {});     //object
    console.log(typeof []);     //object
    console.log(typeof new Date());     //object
    console.log(typeof new RegExp);     //object
```
#### instanceof
检测引用类型一个比较好的方法是使用instanceof运算符。
```
    //检测日期
    if (value instanceof Date) {
        console.log(value.getTime());
    }

    //检测数组
    if (value instanceof Array) {
        value.forEach(() => {});
    }
```
但是instanceof有两个缺点：
- instanceof不仅检测它作用的对象的构造函数，还是检测该对象的原型链的构造函数，只要有一个符合，就会返回true；比如：
```
    var arr = new Array();
    var obj = new Object();
    console.log( arr instanceof Object);    //true
    console.log( arr instanceof Array);    //true
```
在判断Object类型时，就会不准确。
- instanceof有一个严重的限制，就是在嵌套iframe中的页面中可能出问题，如果判断的数据并不是属于该窗口下的数据，instanceof就会失效，原因在于不同窗口下的window对象不是同一个引用。

#### Object.prototype.toString

Kangax发现绑定某个值为执行环节(this)调用Object原型的toString方法，会精确地返回改值的类型。

```
console.log(Object.prototype.toString.call([])); // "[object Array]"
console.log(Object.prototype.toString.call({})); // "[object Object]"
console.log(Object.prototype.toString.call(new Date())); // "[object Date]"
console.log(Object.prototype.toString.call(5)); // "[object Number]"
console.log(Object.prototype.toString.call(['abc'])); // "[object String]"
console.log(Object.prototype.toString.call([undefined])); // "[object Undefined]"
console.log(Object.prototype.toString.call([null])); // "[object Null]"
console.log(Object.prototype.toString.call(function() {})); // "[object Fuction]"
```
该方法在任何情况下都是可靠地，已经被大多数javascript类库所采用，对于原始类型typeof更简单一些，和该方法效果一致。

对于数组，es5引入了Array.isArray()方法来判断，IE9+、Firefox4+、Safari5+、Opera10.5+和Chrome都实现了该方法。

对于函数，typeof也会返回'function'，这也是一个可靠的判断方法。但是在IE8和更早版本的IE中，typeof检测DOM方法都会返回'object'。
```
    // IE 8和更早的IE
    console.log(typeof document.getElementById);    //'object'
```
如果不兼容IE8，就不用考虑这个问题了。对于DOM方法，DOM有明确定义，如果有该成员则意味着它是一个函数，所以我们经常采用`in`操作符来判断：
```
    // 检测DOM方法
    if ('querySelector' in document){
        image = document.querySelector('img');
    }
```
### 总结

String、Number、Boolean、undefined、function(DOM方法除外)可以采用typeof来判断。
引用类型采用Object.prototype.toString方法判断。