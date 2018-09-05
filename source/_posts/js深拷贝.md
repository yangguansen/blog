---
title: js深拷贝
date: 2018-03-31 10:37:45
tags:
	- JavaScript
---

> 前言

js数据类型分为基本数值类型（Undefined，Boolean，number, string）和引用类型(array,object,null)，基本数值类型是在栈中的，引用类型是在堆中的，因此对象是在堆中占据内存的，因此对象的改变，其实是指针指向的对象的改变，因此引用这个对象的对象都会改变。

> 代码

```
const obj1 = {
	name: 'sam',
	sex: 'man',
	love: [ '1', '2', '4' ]
};

// const deepCopy = function ( obj ) {
// 	let newobj;
//
// 	console.log( typeof obj );
// 	switch ( typeof obj ) {
// 		case 'undefined':
// 			break;
// 		case 'string':
// 			newobj = obj + '';
// 			break;
// 		case 'number':
// 			newobj = obj - 0;
// 			break;
// 		case 'boolean':
// 			newobj = obj;
// 			break;
// 		case 'object':
// 			if ( obj === null ) {
// 				newobj = null;
// 			} else {
// 				if ( obj instanceof Array ) {
// 					newobj = [];
// 					for ( let i = 0; i < obj.length; i++ ) {
// 						newobj.push( deepCopy( obj[ i ] ) );
// 					}
// 				} else {
// 					newobj = {};
// 					for ( let key in obj ) {
// 						newobj[ key ] = deepCopy( obj[ key ] );
// 					}
// 				}
// 			}
// 			break;
// 		default:
// 			newobj = obj;
// 	}
//
// 	return newobj;
// };

const deepCopy = function ( obj ) {
	return JSON.parse( JSON.stringify( obj ) );
};

const obj2 = deepCopy( obj1 );

console.log( obj2 );

obj2.love.push( '3', '3' );

console.log( obj1 );
```

以上是深拷贝方法，对象复制后，新对象的改变不会相互影响。

（完）