---
title: HTML id标签会自动生成全局变量
date: 2016-10-18 17:07:53
tags:
	- JavaScript
---
最近在看《你不知道的JavaScript》这本书，里面写道带有ID的HTML标签会自动生成同名的全局变量，我测试了下，上代码:
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<div id="foo"></div>
	<script>
		console.log(foo);
		foo.innerHTML = "123";
	</script>
</body>
</html>```
测试结果：带有ID的HTML标签会自动生成同名的全局变量。
记得去年遇到过这样的问题，当时我还很纳闷，现在看来，也就释然了。所以以后还是少用id属性吧。