---
title: JS模拟鼠标点击坐标
date: 2016-04-26 12:44:52
tags:
	- JavaScript
---
今天一个单子的需求是这样，要模拟鼠标点击坐标，于是百度了下如何实现，看到有个帖子给出了方法，于是实验了下，可以实现，又百度了下关于其的解释，这里写出来记录下：
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style>
	#div1{
		position: absolute;
		width: 200px;
		height: 200px;
		background-color: red;
		color:white;
		text-align: center;
    	line-height: 200px;

	}
	#div2{
		position: absolute;
		left: 300px;
		width: 200px;
		height: 200px;
		background-color: blue;
		color:white;
	    text-align: center;
    	line-height: 200px;

	}

	</style>
</head>
<body>
	<div id="div1">区域1</div>
	<div id="div2">区域2</div>
  <script>
	function imitateClick(oElement, iClientX, iClientY) {  	//定义模拟点击事件
        var oEvent;  
        
        oEvent = document.createEvent("MouseEvents");  
        oEvent.initMouseEvent("click", true, true, document.defaultView, 0, 0, 0,   
                                iClientX, iClientY/*, false, false, false, false, 0, null*/);   
        oElement.dispatchEvent(oEvent);          
    }  
    
    var div1 = document.getElementById('div1');	//区域1
    var div2 = document.getElementById('div2');	//区域2
    div1.onclick = function(event) {  //区域1点击方法	
        alert("点击区域1 (" + event.clientX + "," + event.clientY + ")");  
    }; 
    div2.onclick = function(event) {  //区域2点击方法
        alert("点击区域2 (" + event.clientX + "," + event.clientY + ")");  
    }; 
    
    imitateClick(div1, 100, 100);  //触发模拟点击div1（100,100）的坐标
  </script>
            
       
</body>
</html>
```
这个网站里介绍了关于他的解释，然后发现这个网站关于web介绍的挺全的，可以作为前端学习的一个工具网站：https://developer.mozilla.org/en-US/docs/Web/API/Document/createEvent