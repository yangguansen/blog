---
title: canvas合成图片 / 截屏思路
date: 2016-06-15 15:10:36
tags:
	- JavaScript
---
现在很多手机上的应用都有这样一个需求：选择背景图片，然后上传自己头像，然后合成一张图片分享出去。思路是这样的：
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style>
	canvas{
		width:200px;
		height:200px;
		position: absolute;
	    left: 10px;
    	top: 10px
	}
	#sc{
		border: 10px solid #000;
		width: 200px;
		height: 200px;
	}
	</style>
</head>
<body>
	<input type="file" id="photo">
	<canvas id="cvs"></canvas>
	<img id="sc"></div>
	<input type="button" value="生成" id="scbtn">
	<script>
		document.getElementById('photo').addEventListener('change',function(e){
			img = new Image();
			var files = this.files;		
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
				var c = document.getElementById("cvs");
				var ctx = c.getContext("2d");
				ctx.drawImage(img,10,10);
			}
		});
		var btn = document.getElementById("scbtn");
		btn.onclick = function(){
			var tt = document.getElementById("cvs").toDataURL("image/png");
			document.getElementById('sc').setAttribute("src", tt);
		}
	</script>
</body>
</html>
```
大致思路就是：每上传一张图片，就用canvas的drawImage()方法绘制出来图像，图像会自动叠加，然后生成的时候，用toDataURL方法转换成一张新图片添加到DOM里，这个过程中可以添加拖拽效果之类的