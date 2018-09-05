---
title: JS设置超过设置的文本长度显示的内容
date: 2016-04-26 15:17:43
tags:
	- JavaScript
---
当页面上一行字太多，我们需要把超出长度的内容显示为：“内容...”这种形式。如果是字母符号，2个为一个汉字长度,先是获取内容长度：
```
function getStrChineseLength(str)
{
	if(typeof(str)=="undefined")return str;
	str = str.replace(/[ ]*$/g,"");
	var w = 0;
	for (var i=0; i<str.length; i++) {
	 var c = str.charCodeAt(i);
	 if ((c >= 0x0001 && c <= 0x007e) || (0xff60<=c && c<=0xff9f)) {
	   w++;
	 }else {
	   w+=2;
	 }
	} 	
	var length = w % 2 == 0 ? (w/2) : (parseInt(w/2)+1) ;
	return length;
}
getStrChineseLength("都还好滴啊会fds...  fse滴啊会独爱");
```

限制之后显示的文本内容：
```
function getStrChineseLen(str,len)
{
	if(typeof(str)=="undefined")return str;
	var w = 0;
	str = str.replace(/[ ]*$/g,"");
	if(getStrChineseLength(str)>len){
		for (var i=0; i<str.length; i++) {
			 var c = str.charCodeAt(i);
			 if ((c >= 0x0001 && c <= 0x007e) || (0xff60<=c && c<=0xff9f)) {
			   w++;
			 }else {
			   w+=2;
			 }
			 if(parseInt((w+1)/2)>len){
				return str.substring(0,i-1)+"...";
				break;
			 }
		 
		} 
	}
	return str;
};
```