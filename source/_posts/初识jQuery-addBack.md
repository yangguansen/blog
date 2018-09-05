---
title: 初识jQuery addBack()
date: 2016-04-28 23:29:06
tags:
	- jQuery
---
干了前端大半年，今天第一次在stackoverflow上见到了addBack()方法，刚开始还以为是别人定义的方法，后来一搜才知道是jQuery中定义的方法，找了网上的demo，记录下，分享之，下面上代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<style>
	 <style>
p, div { margin:5px; padding:5px; }
.border { border: 2px solid red; }
.background { background:yellow; }
.left, .right { width: 45%; float: left;}
.right { margin-left:3%; }
    </style>
	</style>
</head>
<body>
	<div class="left">
  <p><strong>Before <code>addBack()</code></strong></p>
  <div class="before-addback">
    <p>First Paragraph</p>
    <p>Second Paragraph</p>
  </div>
</div>
<div class="right">
  <p><strong>After <code>addBack()</code></strong></p>
  <div class="after-addback">
    <p>First Paragraph</p>
    <p>Second Paragraph</p>
  </div>
</div>
 
<script>
$("div.left, div.right").find("div, div > p").addClass("border");
 
// First Example
$("div.before-addback").find("p").addClass("background");
 
// Second Example
$("div.after-addback").find("p").addBack().addClass("background");
</script>
</body>
</html>
```
我理解到的意思就是：$("div.after-addback")用A表示，通过A找到的p标签$("div.after-addback").find("p")用B表示，那么$("div.after-addback").find("p").addBack()返回的对象就是A+B，如果有错的地方，还请指正。