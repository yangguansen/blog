---
title: 'background-attachment:fixed应用'
date: 2015-12-07 23:28:20
tags:
	- CSS
---
设置为fixed属性，背景相对于屏幕窗口固定，然后如果有一张全屏的图片，再来一张全屏的图片，就可以看到与平时滚动屏幕不同的切换图片。代码
CSS部分：
```
html, body,.content{
	height: 100%;
}
h2, body { margin: 0;}
.fixed-bg { 
	position: relative;
	background-size: cover;
	background-attachment: fixed;
	height: 100%;
	background-position: center center;
}
.bg-1 { 
	 background-image: url("images/cd-background-1.jpg");
}
.container { 
	padding: 23% 1%;
   	background-color: #C7ABAB; 
   	height: 100%;
}
.bg-2 { 
	background-image: url("images/cd-background-2.jpg");
}
```

HTML部分：
```
<div class="main content">
	<div class="fixed-bg bg-1">
		<h2>此处是图片</h2>
	</div>
	<div class="fixed-bg bg-2">
		<h2>又一张图片</h2>
	</div>
</div>
```

由此可以想到：如果图片之间加上内容 ，就会是比较新鲜的滚动方式：
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Document</title>
	<style type="text/css">
	html, body,.content{
		height: 100%;
	}
	h2, body { margin: 0;}
	.fixed-bg { 
		position: relative;
		background-size: cover;
		background-attachment: fixed;
		height: 100%;
		 background-position: center center;
	}
	.bg-1 { 
		 background-image: url("images/cd-background-1.jpg");
	}
	.container { padding: 23% 1%;
   	 	background-color: #C7ABAB; 
   	 	height: 100%;}
	.bg-2 { 
		 background-image: url("images/cd-background-2.jpg");
	}

	</style>
</head>
<body>
	<div class="main content">
		<div class="fixed-bg bg-1">
			<h2>此处是图片</h2>
		</div>
		<div class="scrolling-bg"> 
			<div class="container">
				<p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Dolore incidunt suscipit similique, dolor corrupti cumque qui consectetur autem laborum fuga quas ipsam doloribus sequi, mollitia, repellendus sapiente repudiandae labore rerum amet culpa inventore, modi non. Illo quod repellendus alias? Cum rem doloremque adipisci accusantium? Saepe, necessitatibus!</p>
			</div>
		</div>
		<div class="fixed-bg bg-2">
			<h2>又一张图片</h2>
		</div>
		<div class="scrolling-bg"> 
			<div class="container">
				<p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Dolore incidunt suscipit similique, dolor corrupti cumque qui consectetur autem laborum fuga quas ipsam doloribus sequi, mollitia, repellendus sapiente repudiandae labore rerum amet culpa inventore, modi non. Illo quod repellendus alias? Cum rem doloremque adipisci accusantium? Saepe, necessitatibus!</p>
			</div>
		</div>
	</div>
</body>
</html>
```