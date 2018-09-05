---
title: HTML input上传图片并预览
date: 2016-06-13 10:26:36
tags:
	- JavaScript
---
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
</head>
<body>
	<input type="file" id="photo">
	<div id="click" style="width: 200px; height: 200px; border: 1px solid #000;"></div>
	<script>
		document.getElementById('photo').addEventListener('change',function(e){
		
		var files = this.files;
		var img = new Image();
		var reader = new FileReader();
		reader.readAsDataURL(files[0]);
		reader.onload = function(e){
			var mb = (e.total/1024)/1024;
			if(mb>= 2){
				alert('文件大小大于2M');
				return;
			}
			img.src = this.result;
			img.style.width = "80%";
			document.getElementById('click').style.width="200px";
			document.getElementById('click').style.height="200px";
			document.getElementById('click').innerHTML = '';
			document.getElementById('click').appendChild(img);
		}
	});
	</script>
</body>
</html>
```