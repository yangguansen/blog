---
title: svg实现动画效果
date: 2017-02-13 09:54:21
tags:
	- svg
---
web动画除了css3的帧动画外，还可以用svg来实现动画效果，例如网上的一个例子[参考链接](http://www.html5tricks.com/demo/html5-svg-fox-run-animation/index.html),这个动画就是用svg的`animate`标签来实现的，同样是通过软件绘制好图形，path标签中，d属性表示路径点坐标，然后使用animate标签来切换路径，animate标签是path标签的子级，使用 `attributeName=“d”`表示切换父标签path的d属性，然后`values`属性则等于要切换的另一个路径图形，多个图形可以使用分号";"来区分，例如一个丑陋的图形变化[参考链接](http://ol5kv623u.bkt.clouddn.com/ceshi3.html)
参考信息：http://www.zhangxinxu.com/wordpress/2014/08/so-powerful-svg-smil-animation/