---
title: js二级联动菜单
date: 2016-07-08 18:02:37
tags:
	- JavaScript
---
二级联动菜单
<!--more-->
```
<html>
<head>
<title>This is a test!</title>
</head>
<body>
<form name="frm">
<select name="s1" onChange="redirec(document.frm.s1.options.selectedIndex)">
<option selected>请选择</option>
<option value="四川">四川</option>
<option value="云南">云南</option>
<option value="陕西">陕西</option>
<option value="重庆">重庆</option>
</select>

<select name="s2" id="fk">
<option value="请选择" selected>请选择</option>
</select>


</form>
<script language="javascript">
//获取一级菜单长度
var select1_len = document.frm.s1.options.length;
var select2 = new Array(select1_len);
//把一级菜单都设为数组
for (i=0; i<select1_len; i++)
{ select2[i] = new Array();}
//定义基本选项
select2[0][0] = new Option("请选择", " ");

select2[1][0] = new Option("成都", "成都");
select2[1][1] = new Option("自贡", "自贡");
select2[1][2] = new Option("攀枝花", "攀枝花");
select2[1][3] = new Option("泸州", "泸州");
select2[1][4] = new Option("德阳", "德阳");
select2[1][5] = new Option("绵阳", "绵阳");
select2[1][6] = new Option("广元", "广元");
select2[1][7] = new Option("遂宁", "遂宁");
select2[1][8] = new Option("内江", "内江");
select2[1][9] = new Option("乐山", "乐山");
select2[1][10] = new Option("南充", "南充");
select2[1][11] = new Option("宜宾", "宜宾");
select2[1][12] = new Option("广安", "广安");
select2[1][13] = new Option("达州", "达州");
select2[1][14] = new Option("资阳", "资阳");
select2[1][15] = new Option("巴中", "巴中");
select2[1][16] = new Option("雅安", "雅安");
select2[1][17] = new Option("眉山", "眉山");
select2[1][18] = new Option("阿坝藏族羌族", "阿坝藏族羌族");
select2[1][19] = new Option("甘孜藏族", "甘孜藏族");
select2[1][20] = new Option("凉山彝族", "凉山彝族");
select2[1][21] = new Option("华蓥", "华蓥");
select2[1][22] = new Option("简阳", "简阳");


select2[2][0] = new Option("昆明", "昆明");
select2[2][1] = new Option("曲靖", "曲靖");
select2[2][2] = new Option("玉溪", "玉溪");
select2[2][3] = new Option("昭通", "昭通");
select2[2][4] = new Option("临仓地区", "临仓地区");
select2[2][5] = new Option("丽江地区", "丽江地区");
select2[2][6] = new Option("思茅地区", "思茅地区");
select2[2][7] = new Option("保山", "保山");
select2[2][8] = new Option("文山壮族苗族", "文山壮族苗族");
select2[2][9] = new Option("红河哈尼族彝族", "红河哈尼族彝族");
select2[2][10] = new Option("西双版纳傣族", "西双版纳傣族");
select2[2][11] = new Option("楚雄彝族", "楚雄彝族");
select2[2][12] = new Option("大理白族",  "大理白族");
select2[2][13] = new Option("德宏傣族景颇族", "德宏傣族景颇族");
select2[2][14] = new Option("怒江傈僳族", "怒江傈僳族");
select2[2][15] = new Option("迪庆藏族", "迪庆藏族");


select2[3][0] = new Option("西安", "西安");
select2[3][1] = new Option("铜川", "铜川");
select2[3][2] = new Option("宝鸡", "宝鸡");
select2[3][3] = new Option("咸阳", "咸阳");
select2[3][4] = new Option("渭南", "渭南");
select2[3][5] = new Option("延安", "延安");
select2[3][6] = new Option("汉中", "汉中");
select2[3][7] = new Option("榆林", "榆林");
select2[3][8] = new Option("安康", "安康");
select2[3][9] = new Option("商洛", "商洛");

select2[4][0] = new Option("重庆", "重庆");
//联动函数
function redirec(x)
{
	document.getElementById("fk").innerHTML = '';
	// var temp = document.frm.s2;
	for (i=0;i<select2[x].length;i++)
	{ document.frm.s2.options[i]=new Option(select2[x][i].text, select2[x][i].value);}
	document.frm.s2.options[0].selected=true;
}
</script>
</body>
</html>
```