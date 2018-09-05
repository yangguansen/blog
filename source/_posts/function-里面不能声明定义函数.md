---
title: '$(function(){})里面不能声明定义函数'
date: 2015-12-18 12:12:37
tags:
	- JavaScript
---
关于JS变量作用域的使用
<!--more-->
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
    <div onclick="abc()">测试测试</div>                           
<script>
    
$(function(){   
     function abc(){
       alert(123)
     }
})
   
</script>
</body>
</html>
```

在ready函数里这样写会弹出函数未找到，如果写成这样：
```
(function(){
    $("div").on("click", function(){
      alert(123)
    })    
})
```

这样会执行成功，或者去掉ready这一层也能执行成功。

百度上搜为什么第一种不能执行，得到的答案差不多就是因为ready是局部函数。点击事件是在全局里调用，但是我的疑问就是我触发点击事件也是在ready加载完之后执行的，也应该是在ready作用域中啊。希望有想法的朋友可以回复我

先记录下来，目前的结论是：ready里不能声明函数。



2015.12.18 13点16分编辑：
中午趁吃饭时间问了问搞前端的同学，终于明白了其中含义：
div绑定的onclick = abc() 在页面加载 DOM渲染的时候  就会去绑定abc函数，就要去找abc()的函数声明，但是函数声明是在ready里面的，所以并没有找到，也就是没有给abc绑定上函数，所以就算等页面加载完了再去点击，这时候abc已经定型了，就是没找到这个函数声明。
解决办法：
方法一：把ready那一层去掉。
方法二：HTML里不绑定onclick,在JS里写成$("div").on("click", function(){})