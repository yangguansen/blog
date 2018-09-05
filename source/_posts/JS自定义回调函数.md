---
title: JS自定义回调函数
date: 2016-09-20 22:24:01
tags:
	- JavaScript
---
对于jQuery的回调函数，想搞懂他的原理，然后又想到了之前在公司的代码框架里就有自定义回调函数：
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<script>
	function a(callback){
		console.log(1);
		console.log(2);
		if(callback){callback();}
	}
	function b(){
		console.log("回调B");
	}
	function c(){
		console.log("回调C");
	}
	a(b);
	a(c);
	</script>
</body>
</html>
```
简单来说，就是传参。记录之