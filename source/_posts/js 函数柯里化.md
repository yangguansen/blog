---
title: js 函数柯里化
date: 2018-03-31 10:39:45
tags:
	- JavaScript
---

> 前言

函数柯里化是一种编程技巧，表现特点是将多个函数，一层一层包裹，执行最外部函数，把参数传给与其相邻的内层函数，递归调用。来看代码：

> 代码

```
var testCurried = function ( a ) {
	return function ( b ) {
		return function ( c ) {
			return function ( d ) {
				console.log( a + b + c + d );
			}
		}
	}
}

testCurried( 1 )( 2 )( 3 )( 4 );  // 10

```

如果要锁定前三个参数，第四个参数灵活化，那就把前三个方法的调用赋值给一个方法：

```
var test1 = testCurried( 1 )( 2 )( 3 ); // 返回的是个方法

test1( 4 ); // 10
```

对于最后一层调用函数来说，前三个函数形成了闭包，把参数这个变量传递给了最内层函数，`test1`是`testCurried( 1 )( 2 )( 3 )`的执行结果。

以上便是自己对函数柯里化的简单理解。

（完）