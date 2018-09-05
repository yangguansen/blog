---
title: 加载img图片百分比
date: 2016-08-12 09:56:24
tags:
	- JavaScript
---
首先在html中图片要用img标签，div的background在这里不适用，接着就是加载img图片，根据图片加载状态来计算百分比：
```
var jiazai = 0;
var imgs = document.getElementsByTagName("img");
var imgLen = imgs.length;
for(var i = 0; i < imgLen; i++){
    if(imgs[i].complete){    //如果缓存里有这张图片，就是触发complete
        jiazai++;
        var baifenbi = Math.floor(jiazai/imgLen*100);
        setBaiNum(baifenbi);
        continue;
    }
    imgs[i].onload = function(){    //图片加载
        jiazai++;
        var baifenbi = Math.floor(jiazai/imgLen*100);
        setBaiNum(baifenbi);        
    }
}
function setBaiNum(bai){
    $("#baifen_txt").text(bai);   //设置百分数到DOM上
    if(bai == 100){		//图片加载完毕
        setTimeout(function(){
          //  显示视图;
        }, 1000);
        
    }
}
```