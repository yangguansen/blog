---
title: jQuery获取和设置disabled属性、背景图片路径
date: 2015-12-11 15:22:55
tags:
	- jQuery
---
之前对于这个独特的disabled属性获取和设置很混乱，今天项目中用到了，用attr不能实现，于是多次试验得出：
获取disabled属性用prop
```
$("#basic_key").prop("disabled")
```
以上会返回true或false.

然后设置disabled是attr，重点是后面的一个参数不加引号：
```
$("#basic_key").attr("disabled",'false')		//false加引号是错误的~！
$("#basic_key").attr("disabled",false)	//不加引号才正确
```
另外，背景图片路径的获取是：
```
$(".mcht_tdcode").css("background-image")
```
设置图片路径是：
```
$(".mcht_tdcode").css("background-image","url(template/images/p9.jpg)");
```