---
title: createjs第二课----转换坐标+碰撞事件
date: 2016-04-11 21:53:58
tags:
	- createJS
---
createjs转换坐标+碰撞事件
<!--more-->
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
	var stage, arm; 
	window.onload = function(){
		stage = new createjs.Stage("demoCanvas");

		target = stage.addChild(new createjs.Shape());
		target.graphics.beginFill("red").drawCircle(0, 0 ,45)
			.beginFill("white").drawCircle(0,0,30)
			.beginFill("red").drawCircle(0,0,15);
		target.x = 100;
		target.y = 180;

		arm = stage.addChild(new createjs.Shape());
		arm.graphics.beginFill("black").drawRect(-2,-2,100,4)
			.beginFill("blue").drawCircle(100,0,8);
		arm.x = 180;
		arm.y = 100;

		createjs.Ticker.on("tick", tick);
	}
	function tick(){
		arm.rotation += 5;
		target.alpha = 0.2;
		var pt = arm.localToLocal(100, 0, target);
		if (target.hitTest(pt.x, pt.y)){target.alpha = 1;}
		stage.update();
	}
	</script>

</body>
</html>
```