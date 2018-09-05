---
title: 解决“webkitAnimationEnd入场动画切换”在安卓机兼容性问题
date: 2016-12-16 09:01:19
tags:
	- JavaScript
---
问题描述： 
    在手机端H5中，当一个图片入场动画执行完毕后，我们想让它循环执行另一个动画，就是用webkitAnimationEnd这个方法，比如代码是这样的：
```javascript
$(document).on("webkitAnimationEnd", ".p54", function(){
    $(".p54").removeClass('rollIn').addClass('pulse infinite').css({
        "animation-delay" : "0s",
        "-webkit-animation-delay" : "0s",
    })
});
```
这个写法在IOS机型中没有问题，但是在安卓机上并不会执行下一个pulse这个动画。
解决方案是：
在p54这个标签外再套一层div,比如命名为p54_parent,当p54动画执行完后，让p54_parent执行下一个动画，代码是这样的：
```javascript
$(document).on("webkitAnimationEnd", ".p54", function(){
    $(".p54_parent").addClass('pulse infinite')；
});
```
这样，就可以完美解决这个兼容性问题，安卓机也没有问题



