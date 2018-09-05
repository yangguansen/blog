---
title: php之CURL获取跨域信息
date: 2016-06-03 11:12:24
tags:
	- php
---
让给一个客户做一个接口转换，提交的数据以前是在本站的域名下，现在要到客户的后台，这就要涉及到跨域的问题，我本站用的JSONP，但是客户那边后台反馈回来的格式是json的，不是jsonp的，没有用函数包裹，无奈就需要使用php中转跨域，后台不会存在跨域问题，然后又搜了搜，用curl请求服务端数据，代码如下：
```
$arr = array(
        "createDate" => $_GET['createDate'],
        "name" => $_GET['name'],
        "gender" => $_GET['gender'],     //接到的前端数据
    );
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, '链接');
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, $arr);
$result = curl_exec($ch);//向服务器发送数据，返回的是服务端返回的数据
$temp=explode(":",$result);
$num=explode("}",$temp[1]);
$fasong = array("status" => $num[0]);
$json_fasong = json_encode($fasong);
echo $_GET['jsonp_callback'] . "(" . $json_fasong . ")";用函数包裹返回给前端
curl_close($ch);
```