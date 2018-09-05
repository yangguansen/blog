---
title: 使用七牛云存储上传照片js-sdk
date: 2017-04-11 11:00:29
tags:
	- JavaScript
---
七牛云存储为个人提供10G的免费云空间，对个人来说，可以作为上传照片文件的云空间还是蛮不错的。官方提供了多种服务器语言sdk,下面就介绍下如何使用js-sdk配合php实现上传照片功能。
1、首先需要注册一个七牛账户，并获取到accessKey和secretKey，可以在空间设置中查看。

2、需要一门服务器端语言像七牛服务器通信获取token,并将token返回到前端，这里以php为例获取token:
```php
<?php
require_once  './autoload.php';
use Qiniu\Auth;

$bucket = 'save';	//空间名，不是空间域名
$accessKey = '账户accessKey';
$secretKey = '账户secretKey';

$auth = new Auth($accessKey, $secretKey);
$upToken = $auth->uploadToken($bucket);

echo $upToken;
?>
```
php依赖php SDK,需要到七牛官方下载七牛php SDK并引入。初始化实例便可获取token。

3、
前端这里以formData的方式提交照片：
七牛官方规定需要提交照片文件与token值，并以form表单的方式提交：
```
<input name="token" id="token" type="hidden" value="">
<input id="userfile" name="file" type="file" />
```
token的值需要按照步骤2先获取token。提交地址是七牛提供的url,不同地区提交到不同域名，例如我的demo是提交到：[链接](http://up-z2.qiniu.com)，错误的url七牛会报错，并提醒你正确提交的地址。
详情可见demo：http://www.sanmuweb.com/qiniu/index.html