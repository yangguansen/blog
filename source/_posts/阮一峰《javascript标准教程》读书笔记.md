---
title: 阮一峰《javascript标准教程》读书笔记
date: 2016-05-31 15:28:06
tags:
---
最近看到阮大神的《javascript标准教程》这本书，其中的内容非常的基础，于是打算过一遍这本书，其中学到的新技巧也会自己记录更新到这里。
PS：阮大神的博文真的是简单易懂，各种术语翻译成大白话真是方便了我等小白。
<!--more-->
1、查看对象中的所有键名：
```
var o = {
	m1: 1,
	m2: 2
}
console.log(Object.keys(o)); 	//["m1", "m2"]
```

2、删除对象中的属性：
```
var o = {
	m1: 1,
	m2: 2
}
delete o.m1;
console.log(Object.keys(o)); 	//["m2"]
```

3、in可以遍历对象中的属性，和hasOwnProperty()的区别是：in会遍历对象继承的属性，hasOwnProperty只遍历对象本身的属性：
```
function Person(name){
	this.name = name;
}
Person.prototype.describe = function() {
	return "Name: " + this.name;
};
var person = new Person('Jane');

for(var key in person){
	if(person.hasOwnProperty(key)){
		console.log(key);	//name
	}
	
}
```

```
function Person(name){
	this.name = name;
}
Person.prototype.describe = function() {
	return "Name: " + this.name;
};
var person = new Person('Jane');
for(var key in person){		
	console.log(key);	//name, describe	
}
```