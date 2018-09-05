---
title: vue-cli router history 服务端配置
date: 2017-12-10 20:29:45
tags:
	- 总结
---

> 项目背景

最近在做一个项目，用vue-cli搭建，build之后，传到服务器上，手动输入二级子路由，页面显示404。

> 问题原因

手动输入链接，都是要走服务器http请求的，而vue build之后是一个单页应用，所有的路由配置都是通过index.html用js去解析，所以输入的子路由通过服务器解析后并未找到该文件，因此报错404。

> 解决方案

既然要通过index.html去跳转，我们就要把所有的手动输入子路由都要转向到index.html文件，由于我用的是apache，vue-router也说明了history模式需要后台去配置，因此我们在httpd.conf文件中要开启apache的rewrite模块，然后设置允许通过`.htaccess`文件来设置`override`规则。

httpd.conf文件：
```
<Directory "/home/sam">
···

AllowOverride All

···
</Directory>
```

然后在项目根目录中添加`.htaccess`文件，文件中写入vue-router文档中的配置：
```
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

`RewriteBase /`指向的是服务器根目录，我的项目是在二级目录，即/jxvote/，因此要把`RewriteBase`设置为`RewriteBase /jxvote/`, `RewriteRule . /index.html [L]`设置为`RewriteRule . /jxvote/index.html [L]`。

最后再把router中的`base`设置为：`base:'/jxvote/'`。

这样就可以使用history模式下，手动输入子路由打开页面了。

(完)