---
title: blob对象之上传图片本地预览
date: 2016-09-07 11:07:03
tags:
	- JavaScript
---
看到网上有人说blob上传图片到服务器，相对Base64来说是个更好的选择，于是自己尝试了下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<input id="choose" type="file" value="选择">
	<img id="img" src="" alt="" height="100px">
	<script>
		var choose = document.getElementById("choose");
		var img = document.getElementById("img");
		choose.onchange = function (e){
			var f = e.target.files[0];
			var src = window.URL.createObjectURL(f);
			img.src = src;
		}
	</script>
</body>
</html>
```
以上即可实现了选择图片本地预览。