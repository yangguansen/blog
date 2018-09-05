---
title: js时间格式化
date: 2016-04-26 14:33:01
tags:
	- JavaScript
---
时间格式化
<!--more-->
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	
</head>
<body>
	
  <script>
  function time(){


	var date=new Date();
	var y=date.getFullYear();
	var m=date.getMonth()+1;
	if(m<10)m="0"+m;
	var d=date.getDate();
	if(d<10)d="0"+d;
	var h=date.getHours();
	if(h<10)h="0"+h;
	var i=date.getMinutes();
	if(i<10)i="0"+i;
	var s=date.getSeconds();
	if(s<10)s="0"+s;
	var mmm = y+"-"+m+"-"+d+" "+h+":"+i+":"+s;
	console.log(mmm); 
  };
  time();
  </script>
            
       
</body>
</html>
```