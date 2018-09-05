---
title: CSS打字机效果
date: 2016-06-01 16:11:39
tags:
	- CSS
---
运用css中的border闪烁来模拟打字机效果，可用性不是很高，适合内容只有一行的字。
<!--more-->
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style>
	@keyframes typing { from { width: 0; } }
	@keyframes blink-caret { 50% { border-color: transparent; } }

	h1 { 
		font: bold 200% Consolas, Monaco, monospace;
		border-right: .1em solid;
		width: 16.5em; /* fallback */
		/* width: 30ch; # of chars */
		margin: 2em 1em;
		white-space: nowrap;
		overflow: hidden;
		animation: typing 20s steps(30, end), /* # of steps = # of chars */
		           blink-caret .5s step-end infinite alternate;
	}
	</style>
</head>
<body>
	<h1>Typing animation by Lea Verou.</h1>
</body>
</html>
```