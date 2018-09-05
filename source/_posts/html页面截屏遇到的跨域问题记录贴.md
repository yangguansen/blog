---
title: html页面截屏遇到的跨域问题记录贴
date: 2016-07-21 21:17:09
tags:
	- JavaScript
---
canvas在截屏的时候，先用drawImage再用toDataURL，如果图片的链接中域名不是自己的域名，会报错提示跨域问题，解决办法就是从后端请求图片到前端，把图片转换成64位码，这样就不会有跨域的问题了。贴下主要代码：
```
window.onload = function(){
	convertImgToBase64('图片链接', function(base64Img){
	    div.setAttribute("src", base64Img);
	});
	var canvas;
	var div = document.getElementById("ceshi");
	html2canvas(div,{canvas:canvas}).then(function(canvas){
		document.body.appendChild(canvas);
		canvas.toDataURL();//截屏的图片64位码
	})
}
function convertImgToBase64(url, callback, outputFormat){
    var canvas = document.createElement('CANVAS'),
        ctx = canvas.getContext('2d'),
        img = new Image;
    img.crossOrigin = 'Anonymous';
    img.onload = function(){
        canvas.height = img.height;
        canvas.width = img.width;
        ctx.drawImage(img,0,0);
        var dataURL = canvas.toDataURL(outputFormat || 'image/png');
        callback.call(this, dataURL);
        canvas = null; 
    };
    img.src = url;
}
```
不过应该很少有人遇到这个问题，因为有条件的人，图片都在自己服务器上，可以我这个活儿图片不能放在自己服务器上，所以只能这样搞了