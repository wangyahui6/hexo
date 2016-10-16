---
title: javascript 的封装和继承机制
date: 2016-10-16 21:18:13
categories: web前端
tags: javascript
---
## 封装
所谓封装，就是将一系列相关的属性和方法组合到一起，并留出一个对外的接口。在面向对象的语言中封装的实现就是类。javascript中并没有类的概念，那么javascript是如何实现封装的那？我们可以从最简单的封装说起，对象其实就是一个最简单的封装：
```
{
	tye:"汽车"，
	passengers:4,
	maxspeed:"300km/h"
}
```
<!-- more -->
这是一个交通工具的封装。但是这种方式的复用不方便，我们每次都要重写一遍这个对象，可以用一个函数返回该对象的方式来做改进：
```
function vehicle() {
	return {
		tye:"汽车"，
		passengers:4,
		maxspeed:"300km/h"
	}
}
var vehicle1 = vehicle();
var vehicle2 = vehicle();
```
使用这个函数就可以得到一个vehicle的实例，vehicle1和vehicle2分别是通过函数vehicle生产的实例。但是这两个实例之间找不到任何联系，无法知道vehicle1是由谁构造出来的。javascript创造出了构造函数来解决这个问题：
```
function vehicle() {
	this.type = "汽车";
	this.passengers = 4;
	this.maxspeed = "300km/h";
}
var vehicle1 =new vehicle();
var vehicle2 =new vehicle();
```
构造函数使用new命令来生成实例，函数中的this指向生成的实例对象。javascript提供一个isInstanceOf方法，来判定构造函数和实例之间的关系：
```
vehicle1 isInstanceOf vehicle    //true
vehicle2 isInstanceOf vehicle    //true
```
使用构造函数可以很方便的生成实例，也可以指明构造函数和实例之间的关系，但是构造函数有一个缺点是实例之间不能共享属性和方法，每个实例都生成一份属性和方法，这样都造成内存的浪费。所以javascript为构造函数加入了prototype属性，构造函数生成的实例都继承了构造函数的prototype的属性和方法：
```
vehicle.prototype = {
	color:"red"
}
var vehicle = new vehicle();
vehicle1.color    //red
vehicle1.constructor = vehicle;
```
可以看到vehicle1继承了prototype的color属性。prototype有一个constructor属性，指向构造函数。这样js就完成了封装的设计。
