---
title: css动画执行完换另一个动画
date: 2016-08-12 11:40:58
tags:
	- JavaScript
---
animation有三个事件，在入场动画执行完想换成另一个循环动画时，用webkitAnimationEnd 事件:
```
$(document).on("webkitAnimationEnd", ".p34", function(){
    $(".p34").removeClass('入场动画类')；
    $(".p34").addClass('循环动画类')
}) 
```
拓展：animation共有3个事件：可在需要时使用
开始事件 webkitAnimationStart 
结束事件 webkitAnimationEnd 
重复运动事件 webkitAnimationIteration 
css3的过渡属性transition，在动画结束时，也存在结束的事件：webkitTransitionEnd
transition仅仅有这一个事件