---
title: JS获取URL中的某个参数
date: 2016-07-12 18:39:32
tags:
	- JavaScript
---
```
String.prototype.getUrlValue = function(parm) {
	var reg = new RegExp("(^|&)" + parm + "=([^&]*)(&|$)");
	var r = this.substr(this.indexOf("\?") + 1).match(reg);
	if (r != null) return unescape(r[2]);
	return null;
}
var id = window.location.href.getUrlValue("uid");
```