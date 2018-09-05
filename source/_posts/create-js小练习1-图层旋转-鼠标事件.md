---
title: create.js小练习1----图层旋转+鼠标事件
date: 2016-04-11 21:36:08
tags:
	- createJS
---
最近总想学点H5小动画之类的框架，在网上搜了很多框架都不知道该从何下手，最后锁定cocos2d-JS和create.js。这两个都看了一下入门知识，前者适合中大型游戏开发，后者适合小型游戏和动画，但是createJS已经停止维护了，但是觉得create还简单些，拿来动画入门还是不错的，遂做点练习来入门，等练熟之后再往高处走，揣测着H5应用的流行趋势，希望自己的选择是正确的
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<script src="https://code.createjs.com/easeljs-0.8.2.min.js"></script>
	
</head>
<body>
	
	<canvas id="demoCanvas" width="500" height="300"></canvas>

	<script>
	var stage, circle; 
	stage = new createjs.Stage("demoCanvas");
	circle = stage.addChild(new createjs.Container());
	circle.x = circle.y = 150;

	for (var i = 0; i < 25; i++) {
		var shape = new createjs.Shape();
		shape.graphics.beginFill(createjs.Graphics.getHSL(Math.random()*360, 100, 50)).drawCircle(0,0,30);
		shape.x = Math.random()*300 - 150;
		shape.y = Math.random()*300 - 150;
		circle.addChild(shape);
	}
	createjs.Ticker.on("tick",tick);
	function tick(){
		circle.rotation += 3;
		var l = circle.getNumChildren();
		for (var i = 0 ;i<l;i++){
			var child = circle.getChildAt(i);
			child.alpha = 0.1;
			var pt = child.globalToLocal(stage.mouseX, stage.mouseY);
			if(child.hitTest(pt.x, pt.y)){
				child.alpha = 1;
			}
		}
		stage.update();
	}
	</script>

</body>
</html>
```