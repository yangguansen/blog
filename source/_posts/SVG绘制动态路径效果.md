---
title: SVG绘制动态路径效果
date: 2017-02-13 09:17:36
tags:
	- svg
---
svg动态绘制路径效果，使用snap.js库，近适用于path标签。
<!--more-->
svg标签HTML代码我们可以使用AI软件或者[在线制作](http://editor.method.ac/)，具体svg知识大家可以参考[张鑫旭](http://www.zhangxinxu.com/wordpress/category/graphic/svg-graphic/)，张鑫旭的博客进行基础知识学习。使用软件绘制一条路径后，然后我们导出文件为svg格式，看其代码，复制path标签。例如：
```
<svg id="svg" width="640" height="1136" xmlns="http://www.w3.org/2000/svg">
    <g>
        <path id="svg_1" d="m127.5,13.5c56,60 109,17 108.5,46.5c-0.5,29.5 -111.5,13.5 -119,50c-7.5,36.5 168.5,-6.5 182,43c13.5,49.5 -132.5,6.5 -130.5,57.5c2,51 100,29 99.5,28.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
        <ellipse id="yuan" ry="0" rx="0" id="svg_2" cy="332.5" cx="280.5" stroke-width="1.5" stroke="#000" fill="#FFB27C"/>
    </g>
</svg>
```
然后引入snap.js，这个库可以操控svg DOM，使用方法也比较容易学习，这里我们直接使用，参考一下代码：
```
var svg = Snap("#svg");
var snpg = svg.select("#svg_1");
snpg.attr({
    strokeDashoffset: snpg.getTotalLength(),
    strokeDasharray: snpg.getTotalLength()
});
snpg.animate({
    "stroke-dashoffset": 0
}, 2000, mina.easeout);
```
getTotalLength()可以直接获取路径的长度，这样就直接形成了动态路径效果，这个效果用到了路径的strokeDashoffset和strokeDasharray属性，strokeDasharray表示各个虚线段的长度，strokeDashoffset表示虚线的起始偏移量，原理就是先使用attr方法设置成当前的路径长度，然后再使用animate方法使偏移量变为0，变为0的动画效果就形成了动态路径。
通过以上方法，我们就可以绘制多条path路径，然后通过回调函数一条一条的绘制，形成绘制一个物体的动态效果，复制以下代码可以查看效果：
```
<svg id="svg" width="580" height="400" xmlns="http://www.w3.org/2000/svg">
	<g>
		<path id="svg_6" d="m109.5,67.5l-1.5,258.5" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path id="svg_7" d="m110.5,67.5l89.5,-2.5l-6,260" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path id="svg_8" d="m121.5,92.5c62,-19 64,0 64,0" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path stroke="#000" id="svg_9" d="m123.5,116.5c60,-9.707724 54,-0.225761 54,-0.225761" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" fill="#fff"/>
	  	<path stroke="#000" id="svg_10" d="m126.5,134.5c57,-0.250008 58,0 58,0" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" fill="#fff"/>
	  	<path stroke="#000" id="svg_11" d="m124.5,168.626247c59,13.997647 58,-1.126247 58,-1.126247" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" fill="#fff"/>
	  	<path id="svg_12" d="m125.5,201.5l58.5,7.5" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path stroke="#000" id="svg_13" d="m119.5,231.94206c64,-10.531113 66,0.55794 66,0.55794" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" fill="#fff"/>
		<path stroke="#000" id="svg_14" d="m124.5,255.193568c59,-27.578854 56,0.306432 56,0.306432" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" fill="#fff"/>
	  	<path id="svg_15" d="m155.5,84.5c0,0 -3,157 -3.5,156.5" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path id="svg_16" d="m130.5,271.5l-2.5,51.5" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path id="svg_17" d="m131.5,273.5l47.5,1.5l-3,51" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	  	<path id="svg_18" d="m152.5,274.5c0,0 -2,47 -2.5,46.5" opacity="0.5" fill-opacity="null" stroke-opacity="null" stroke-width="1.5" stroke="#000" fill="#fff"/>
	</g>
</svg>
<script>
	var svg = Snap("#svg");
    var countArr = [];
    function hualou(id){
        var snpg = svg.select("#svg_"+id);
        snpg.attr({     
        opacity:1,   
            strokeDashoffset: 746,
            strokeDasharray: 746
        });
        snpg.animate({
            
            "stroke-dashoffset": 0
        }, 2000, mina.easeout);
    }
    function initLou(id){
        var snpg = svg.select("#svg_"+id);
        snpg.attr("opacity", 0);
    }
    window.onclick= function(){
        for(var j = 0; j < countArr.length; j++){
            clearTimeout(countArr[j]);
        }
        for(var i = 6; i < 19; i++){
	        (function(i){
	            initLou(i);
	            countArr.push(setTimeout(function(){ hualou(i) }, 300+300*(i-6)));
	        })(i)        
	    }
    }
    window.ontouchstart= function(){
        for(var j = 0; j < countArr.length; j++){
            clearTimeout(countArr[j]);
        }
        for(var i = 6; i < 19; i++){
	        (function(i){
	            initLou(i);
	            countArr.push(setTimeout(function(){ hualou(i) }, 300+300*(i-6)));
	        })(i)        
	    }
    }    
</script>
```
（完）