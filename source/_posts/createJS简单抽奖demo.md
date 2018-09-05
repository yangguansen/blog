---
title: createJS简单抽奖demo
date: 2016-05-30 15:47:44
tags:
	- createJS
---
最近在学createJS, 就做个抽奖来练手，这个createjs库是google研发的，可以做小游戏引擎和H5小动画，感觉性能不错，语法也容易理解，但是google团队在2015年就停止维护了，不知道为啥，国内网上关于这个的教程挺少的，唯一的一个中文网就是按照官网翻译的，而且翻译的还特别生硬，我最近在墙外发现个日本的网站有这个库的教程，非常不错。也一直苦于不知道做H5小动画用哪个框架比较好，之前看过cocos2D，工程量太大了，相比之下，createJS非常的轻便。
另外还备注个求区间随机数的代码：
```
function selectfrom (lowValue,highValue){
	var choice=highValue-lowValue+1;
	return Math.floor(Math.random()*choice+lowValue);
}
```

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>	
	<script src="https://code.createjs.com/createjs-2015.11.26.min.js"></script>
	<style>
	</style>
</head>
<body>
	<canvas id="demo" width="1000px" height="400px"></canvas>
	
  <script>
	var stage = new createjs.Stage("demo");
	var container = new createjs.Container();
	var shape = new createjs.Shape();
	var line = new createjs.Shape();
	var text1 = new createjs.Text("手机", "15px Arial", "black");
	var text2 = new createjs.Text("电脑", "15px Arial", "black");
	var text3 = new createjs.Text("汽车", "15px Arial", "black");
	var text4 = new createjs.Text("美女", "15px Arial", "black");
	var tip = new createjs.Text("点击红色区域抽奖", "30px Arial", "green");
	var result = new createjs.Text("", "30px Arial", "red");
	var jiantou = new createjs.Shape();
	var jiangpin = ["手机", "电脑", "汽车", "美女"];
	var num;
	var canClick = true;
	jiantou.graphics.beginFill("green")
		.moveTo(-5,-5).lineTo(5, -5).lineTo(0, 15);
	jiantou.x = 100;
	jiantou.y = 40;
	text1.x = text2.x = -30;
	text1.y = text4.y = -20;	
	text2.y = text3.y = 20;
	text3.x = text4.x = 10;	
	container.x = 100;
	container.y = 100;	
	tip.x = result.x = 200;
	tip.y = 50;
 	result.y = 80;
	shape.graphics.beginFill("red").drawCircle(0,0,50);
	line.graphics.beginStroke("#000").setStrokeStyle(2);
	line.graphics.moveTo(-50,0).lineTo(50,0).moveTo(0, 50).lineTo(0,-50);
	stage.addChild(container, jiantou, tip, result);
	container.addChild(shape, line, text1, text2, text3, text4);
	createjs.Ticker.setFPS(100);	
	createjs.Ticker.addEventListener("tick", stage);	
	stage.addEventListener("mousedown", getNum);
	function getNum(){
		if(canClick){
			canClick = false;
			initCircle();
			num = Math.floor(Math.random()*3+1);		
			var rotateReg = Math.floor(Math.random()*(num*90-(num-1)*90+1)+(num-1)*90) + 360*3;
			console.log(num,jiangpin[num-1], rotateReg);
			rotateCircle(rotateReg);
		}
	}
	function rotateCircle(rotateReg){
		createjs.Tween.get(container)
			.to({rotation:rotateReg}, 5000, createjs.Ease.quadOut).call(setResult);
	}
	function initCircle(){
		container.rotation = 0;
	}
	function setResult(){
		result.text = "恭喜你得到：" + jiangpin[num-1];
		canClick = true;
	}
  </script>             
</body>
</html>
```