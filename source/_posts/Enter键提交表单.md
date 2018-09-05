---
title: Enter键提交表单
date: 2015-12-03 17:22:36
tags:
	- JavaScript
---
input type="submit"在360浏览器上不能提交   用了这个
```
<input type="button" class="btn btn_green btn_active btn_block btn_lg reg" value="注 册">
```

```
$(".input").keydown(function(e){	//class为input的最后一个
	var e = e || event,	//event是兼容IE浏览器
	 keycode = e.which || e.keyCode;		//e.keyCode是兼容IE浏览器，e.which是获取按键编码
	 if (keycode==13) {
	 $(".reg").trigger("click");	如果input上设置了点击事件，就要注释掉这句，不然会重复提交两次
	
	 }
});
```