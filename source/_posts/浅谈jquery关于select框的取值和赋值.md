---
title: 浅谈jquery关于select框的取值和赋值
date: 2015-12-18 14:17:51
tags:
	- jQuery
---
```
jQuery("#select_id").change(function(){}); // 1.为Select添加事件，当选择其中一项时触发     
var checkValue = jQuery("#select_id").val(); // 2.获取Select选中项的Value  
var checkText = jQuery("#select_id :selected").text(); // 3.获取Select选中项的Text   
var checkIndex = jQuery("#select_id").attr("selectedIndex");// 4.获取Select选中项的索引值,
或者：jQuery("#select_id").get(0).selectedIndex;  
var maxIndex = jQuery("#select_id :last").attr("index");  // 5.获取Select最大的索引值,
或者：jQuery("#select_id :last").get(0).index;


jQuery("#select_id").get(0).selectedIndex = 1; // 1.设置Select索引值为1的项选中  
jQuery("#select_id").val(4);  // 2.设置Select的Value值为4的项选中
$("#select_id").attr("value","Normal“);
$("#select_id").get(0).value = value;
//根据select的显示值来为select设值
var count=$("#select_id").get(0).options.length;
for(var i=0;i<count;i++){
if($("#select_id").get(0).options[i].text == text)  
{
$("#select_id").get(0).options[i].selected = true;          
break;  
}  
}


jQuery("#select_id").append("<option value='新增'>新增option</option>"); // 1.为Select追加一个Option(下拉项)   
jQuery("#select_id").prepend("<option value='请选择'>请选择</option>"); // 2.为Select插入一个Option(第一个位置)  
jQuery("#select_id").get(0).remove(1);  // 3.删除Select中索引值为1的Option(第二个)  
jQuery("#select_id :last").remove();  // 4.删除Select中索引值最大Option(最后一个)   
jQuery("#select_id [value='3']").remove(); // 5.删除Select中Value='3'的Option   
jQuery("#select_id").empty();   // 6.清空下拉列表
```