---
title: HTML5手势解锁
date: 2016-07-21 21:22:41
tags:
	- JavaScript
---
手机端H5中，实现仿手势解锁，用户按照图标滑出某个图形，即可解锁，效果可见[链接](http://cq.sina.com.cn/zt/2016/07/wltkzz.html)
其原理就是：把手势图片分成很多碎段，在touchmove中监听每个触摸坐标是否击中分段图片的开始结尾坐标，如果击中结尾坐标，即把那一段图片显示出来，代码如下：
```
<html class="hb-loaded ks-webkit537 ks-webkit ks-chrome39 ks-chrome">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=GBK">
<meta charset="gb2312">
<title></title>
<meta name="format-detection" content="telephone=yes">
<meta name="viewport" content="width=device-width,user-scalable=no,initial-scale=1, maximum-scale=1">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="HandheldFriendly" content="true">
<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<style>
 body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,pre,code,form,fieldset,legend,input,textarea,p,blockquote,iframe,section{margin:0;padding:0;}
body{font-family:"microsoft yahei";-webkit-text-size-adjust:none;color:#333333;
background:#1f3693 url(http://hubei.sinaimg.cn/ltzt/2014/2014wbzy/zzth5/bg.jpg) no-repeat center top; background-size:cover;}
html{width:100%; height:100%;}
.startpage{width:100%;height:100%;}
.startpage .cont{width:100%;position:relative;}
.startpage .cont .atmain{width:189px;height:188px;position:absolute;top:65%;left:50%;margin:-94px 0 0 -94px;background:url(http://hubei.sinaimg.cn/ltzt/2014/2014wbzy/zzth5/line_18.png);opacity:0;}
.startpage .cont .atmain .atcont{width:189px;height:188px;position:relative;}
.startpage .cont .atmain .atcont .bigxx{width:60px;height:50px;position:absolute;z-index:5;background:url('http://hubei.sinaimg.cn/ltzt/2014/2014wbzy/zzth5/linexx1.png');
transform:scale(1.5,1.5);
-ms-transform:scale(1.5,1.5);
-moz-transform:scale(1.5,1.5);
-webkit-transform:scale(1.5,1.5);
}
.startpage .cont .atmain .atcont .bigxx1{left:89px;top:63px;z-index:5;}

.startpage .cont .atmain .atcont .sxx{width:42px;height:36px;position:absolute;background:url(http://hubei.sinaimg.cn/ltzt/2014/2014wbzy/zzth5/linexxx.png);z-index:5;}
.startpage .cont .atmain .atcont .sxx1{left:63px;top:35px;}
.startpage .cont .atmain .atcont .sxx2{left:35px;top:65px;}
.startpage .cont .atmain .atcont .sxx3{left:42px;top:109px;}
.startpage .cont .atmain .atcont .sxx4{left:85px;top:107px;}
.startpage .cont .atmain .atcont .sxx5{left:136px;top:97px;}
.startpage .cont .atmain .atcont .sxx6{left:136px;top:33px;}
.startpage .cont .atmain .atcont .sxx7{left:79px;top:0px;}
.startpage .cont .atmain .atcont .sxx8{left:17px;top:26px;}
.startpage .cont .atmain .atcont .sxx9{left:-1px;top:89px;}
.startpage .cont .atmain .atcont .sxx10{left:35px;top:143px;}
.startpage .cont .atmain .atcont .sxx11{left:97px;top:152px;}
.startpage .cont .atmain .atcont .sxx12{left:147px;top:131px;}

.startpage .cont .atmain .atcont .atline{position:absolute;z-index:3;display:none;}
.startpage .cont .atmain .atcont .line1{width:60px;height:54px;left:76px;top:33px;}
.startpage .cont .atmain .atcont .line2{width:46px;height:48px;left:33px;top:38px;}
.startpage .cont .atmain .atcont .line3{width:46px;height:46px;left:31px;top:83px;}
.startpage .cont .atmain .atcont .line4{width:70px;height:27px;left:33px;top:123px;}
.startpage .cont .atmain .atcont .line5{width:60px;height:40px;left:75px;top:88px;}
.startpage .cont .atmain .atcont .line6{width:67px;height:40px;left:100px;top:115px;}
.startpage .cont .atmain .atcont .line7{width:44px;height:64px;left:138px;top:54px;}
.startpage .cont .atmain .atcont .line8{width:92px;height:50px;left:89px;top:0px;}
.startpage .cont .atmain .atcont .line9{width:76px;height:42px;left:19px;top:0px;}
.startpage .cont .atmain .atcont .line10{width:54px;height:64px;left:0px;top:42px;}
.startpage .cont .atmain .atcont .line11{width:54px;height:70px;left:0px;top:105px;}
.startpage .cont .atmain .atcont .line12{width:56px;height:36px;left:55px;top:152px;}
.startpage .cont .atmain .atcont .line13{width:70px;height:60px;left:113px;top:127px;}

@keyframes wzout
{
0%{opacity:0;}
100%{opacity:1;}
}
@-ms-keyframes wzout 
{
0%{opacity:0;}
100%{opacity:1;}
}
@-moz-keyframes wzout 
{
0%{opacity:0;}
100%{opacity:1;}
}
@-webkit-keyframes wzout
{
0%{opacity:0;}
100%{opacity:1;}
}

@keyframes xxsf
{
0%{
transform:scale(1,1);
-ms-transform:scale(1,1);
-moz-transform:scale(1,1);
-webkit-transform:scale(1,1);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-ms-keyframes xxsf 
{
0%{
transform:scale(1,1);
-ms-transform:scale(1,1);
-moz-transform:scale(1,1);
-webkit-transform:scale(1,1);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-moz-keyframes xxsf 
{
0%{
transform:scale(1,1);
-ms-transform:scale(1,1);
-moz-transform:scale(1,1);
-webkit-transform:scale(1,1);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-webkit-keyframes xxsf 
{
0%{
transform:scale(1,1);
-ms-transform:scale(1,1);
-moz-transform:scale(1,1);
-webkit-transform:scale(1,1);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}

@keyframes xxsf1
{
0%{
transform:scale(1.2,1.2);
-ms-transform:scale(1.2,1.2);
-moz-transform:scale(1.2,1.2);
-webkit-transform:scale(1.2,1.2);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-ms-keyframes xxsf1 
{
0%{
transform:scale(1.2,1.2);
-ms-transform:scale(1.2,1.2);
-moz-transform:scale(1.2,1.2);
-webkit-transform:scale(1.2,1.2);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-moz-keyframes xxsf1 
{
0%{
transform:scale(1.2,1.2);
-ms-transform:scale(1.2,1.2);
-moz-transform:scale(1.2,1.2);
-webkit-transform:scale(1.2,1.2);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}
@-webkit-keyframes xxsf1 
{
0%{
transform:scale(1.2,1.2);
-ms-transform:scale(1.2,1.2);
-moz-transform:scale(1.2,1.2);
-webkit-transform:scale(1.2,1.2);}
100%{transform:scale(0.5,0.5);
-ms-transform:scale(0.5,0.5);
-moz-transform:scale(0.5,0.5);
-webkit-transform:scale(0.5,0.5);}
}

.startpage .cont .atmain .ainm .xxact{
animation:xxsf 1s linear 0s infinite alternate;
-ms-animation:xxsf 1s linear 0s infinite alternate;
-moz-animation:xxsf 1s linear 0s infinite alternate;
-webkit-animation:xxsf 1s linear 0s infinite alternate;
}
.startpage .cont .atmain .ainm .xxact1{
animation:xxsf 1s linear 1s infinite alternate;
-ms-animation:xxsf 1s linear 1s infinite alternate;
-moz-animation:xxsf 1s linear 1s infinite alternate;
-webkit-animation:xxsf 1s linear 1s infinite alternate;
}

.startpage .cont .atmain .ainm .xxact3{
animation:xxsf1 .3s linear 0s infinite alternate;
-ms-animation:xxsf1 .3s linear 0s infinite alternate;
-moz-animation:xxsf1 .3s linear 0s infinite alternate;
-webkit-animation:xxsf1 .3s linear 0s infinite alternate;
}


.startpage .cont .p2act1{
animation:wzout .8s ease-out 1s 1 forwards;
-ms-animation:wzout .8s ease-out 1s 1 forwards;
-moz-animation:wzout .8s ease-out 1s 1 forwards;
-webkit-animation:wzout .8s ease-out 1s 1 forwards;
}

</style>
</head>
<body><!-- SUDA_CODE_START --> 
  
<!-- SUDA_CODE_END -->

<div class="startpage">
  <div class="cont" style="height: 736px;">    
	<div class="atmain p2act1" id="atcont">
	  <div class="atcont ainm">
	    <p class="bigxx bigxx1 point xxact3"></p>
		<p class="sxx sxx1 point xxact"></p>
		<p class="sxx sxx2 point xxact1"></p>
		<p class="sxx sxx3 point xxact"></p>
		<p class="sxx sxx4 point xxact1"></p>
		<p class="sxx sxx5 point xxact"></p>
		<p class="sxx sxx6 point xxact1"></p>
		<p class="sxx sxx7 point xxact"></p>
		<p class="sxx sxx8 point xxact1"></p>
		<p class="sxx sxx9 point xxact"></p>
		<p class="sxx sxx10 point xxact1"></p>
		<p class="sxx sxx11 point xxact"></p>
		<p class="sxx sxx12 point xxact1"></p>

		<p class="atline line1"><img src="img/line1.png"></p>
		<p class="atline line2"><img src="img/line2.png"></p>
		<p class="atline line3"><img src="img/line3.png"></p>
		<p class="atline line4"><img src="img/line4.png"></p>
		<p class="atline line5"><img src="img/line5.png"></p>
		<p class="atline line6"><img src="img/line6.png"></p>
		<p class="atline line7"><img src="img/line7.png"></p>
		<p class="atline line8"><img src="img/line8.png"></p>
        <p class="atline line9"><img src="img/line9.png"></p>
		<p class="atline line10"><img src="img/line10.png"></p>
		<p class="atline line11"><img src="img/line11.png"></p>
		<p class="atline line12"><img src="img/line12.png"></p>
		<p class="atline line13"><img src="img/line13.png"></p>
	  </div>

	</div>
	
  </div>
</div>
<script>
$(function(){

var pointArr = [];
var points;
var start = true;
var end = false;
var tolerance = 30;

var getPointWz = function(){
    var offset,w,h;
    points = $("#atcont .point");
	for(i=0;i<points.length;i++){
	     offset = points.eq(i).offset();
		 w = points.eq(i).width()/2;
		 h = points.eq(i).height()/2;
		 pointArr[i] = [];
		 pointArr[i][0] = parseInt(offset.left) + w;
		 pointArr[i][1] = parseInt(offset.top) + h;
		 pointArr[i][2] = false;
	}
}

var getPointXY = function(x,y,n){
    var tXX,tYY,tFF;
	tXX = x - pointArr[n][0];
	tXX = tXX < 0 ? tXX * -1 : tXX;
	tYY = y - pointArr[n][1];
	tYY = tYY < 0 ? tYY * -1 : tYY;
	tFF = pointArr[n][2];
	return{
	     x : tXX,
	     y : tYY,
	     f : tFF
	}
}

var atobj = document.getElementById('atcont');

atobj.addEventListener('touchstart', function(event) { 
    event.preventDefault();
	getPointWz();
	var x,y,xy;
	start = false;
    var touch = event.targetTouches[0]; 

    x = touch.pageX;
	y = touch.pageY;

	$(".atcont").removeClass("ainm");

	//$("#wzx span").text(x);
	//$("#wzy span").text(y);

	if(!end){
	     xy = getPointXY(x,y,0);
		 x = xy.x;
		 y = xy.y;
		 if(x <= tolerance && y <= tolerance){
		      pointArr[0][2] = true;
		 }
	}
	//$("#fhx span").text(x);
	//$("#fhy span").text(y);
	//$("#fhff span").text(pointArr[0][2]);
}, false);

atobj.addEventListener('touchmove', function(event) { 
    event.preventDefault();
	if(start) return false;
	var x,y,xy;
    var touch = event.targetTouches[0]; 

    x = touch.pageX;
	y = touch.pageY;

	//$("#wzx span").text(x);
	//$("#wzy span").text(y);
	//$("#fhff span").text(pointArr[0][2]);

	if(!end){
	     if(pointArr[0][2] && !pointArr[1][2]){
		      xy = getPointXY(x,y,1);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[1][2] = true;
				   $("#atcont .atline:eq(0)").fadeIn(200);
		      }
			  //$("#fhx span").text(x);
	          //$("#fhy span").text(y);
	          //$("#fhff span").text(pointArr[1][2]);
		 }else if(pointArr[1][2] && !pointArr[2][2]){
		      xy = getPointXY(x,y,2);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[2][2] = true;
				   $("#atcont .atline:eq(1)").fadeIn(200);
		      }
		 }else if(pointArr[2][2] && !pointArr[3][2]){
		      xy = getPointXY(x,y,3);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[3][2] = true;
				   $("#atcont .atline:eq(2)").fadeIn(200);
		      }
		 }else if(pointArr[3][2] && !pointArr[4][2]){
		      xy = getPointXY(x,y,4);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[4][2] = true;
				   $("#atcont .atline:eq(3)").fadeIn(200);
				   $("#atcont .atline:eq(4)").fadeIn(200);
		      }
		 }else if(pointArr[4][2] && !pointArr[5][2]){
		      xy = getPointXY(x,y,5);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[5][2] = true;
				   $("#atcont .atline:eq(5)").fadeIn(200);
		      }
		 }else if(pointArr[5][2] && !pointArr[6][2]){
		      xy = getPointXY(x,y,6);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[6][2] = true;
				   $("#atcont .atline:eq(6)").fadeIn(200);
		      }
		 }else if(pointArr[6][2] && !pointArr[7][2]){
		      xy = getPointXY(x,y,7);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[7][2] = true;
				   $("#atcont .atline:eq(7)").fadeIn(200);
		      }
		 }else if(pointArr[7][2] && !pointArr[8][2]){
		      xy = getPointXY(x,y,8);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[8][2] = true;
				   $("#atcont .atline:eq(8)").fadeIn(200);
		      }
		 }else if(pointArr[8][2] && !pointArr[9][2]){
		      xy = getPointXY(x,y,9);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[9][2] = true;
				   $("#atcont .atline:eq(9)").fadeIn(200);
		      }
		 }else if(pointArr[9][2] && !pointArr[10][2]){
		      xy = getPointXY(x,y,10);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[10][2] = true;
				   $("#atcont .atline:eq(10)").fadeIn(200);
		      }
		 }else if(pointArr[10][2] && !pointArr[11][2]){
		      xy = getPointXY(x,y,11);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[11][2] = true;
				   $("#atcont .atline:eq(11)").fadeIn(200);
		      }
		 }else if(pointArr[11][2] && !pointArr[12][2]){
		      xy = getPointXY(x,y,12);
		      x = xy.x;
		      y = xy.y;
		      if(x <= tolerance && y <= tolerance){
		           pointArr[12][2] = true;
				   $("#atcont .atline:eq(12)").fadeIn(200);
				   end = true;
		      }
		 }
	}
}, false);

atobj.addEventListener('touchend', function(event) { 
    event.preventDefault();
	start = true;

	$(".atcont").addClass("ainm");

	if(!end){
	     $("#atcont .atline").fadeOut(200);
	}else{
	     setTimeout(function(){
		      $(".startpage").fadeOut(800);
			  setTimeout(function(){
			       $("#wbzymain").fadeIn(500);
			  }, 850);
		 }, 1000);
	     //$(".banner span").text('搞定');
	}
}, false);

})
</script>
</body>
</html>
```