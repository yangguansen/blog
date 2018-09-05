---
title: nginx反向代理和负载均衡配置
date: 2017-09-03 14:44:30
tags:
	- 总结
---

> nginx介绍

nginx作为静态服务器，只能解析静态文件，像php文件就属于动态文件，所以无法打开php文件，因为nginx要想解析php文件，须配合php-fpm使用，nginx当需要解析php文件的时候就会把解析任务交给php-fpm服务，php-fpm再把解析结果返给nginx,最终返回给用户。最近研究了下nginx，使用它的反向代理和负载均衡来解决一些常见问题，下面记录一下。

> 反向代理

现在越来越多的前后端分离已经不只是代码上的分离，就连服务器也各自使用，所以前后端通信就会有跨域的现象出现，以往解决跨域就是使用jsonp，但是目前越来越多的人使用nginx反向代理，即前端请求不直接发给后端，而是发到nginx服务器上，nginx再根据你的配置进行转发，由nginx来访问后端地址，相当于是一个中介。下面是配置示例：

```
server
{
    #监听端口
    listen 81;
    
    #域名
    server_name 120.27.104.77;

    index index.html index.htm index.php;
    
    #站点目录
    root /home/nginx;
    
    location /three/
    {
        proxy_pass http://120.27.104.77:80/three/;
        index index.html index.php;
        proxy_set_header Host $host;
        proxy_buffer_size 64k;
        proxy_buffers   32 32k;
        proxy_busy_buffers_size 128k;
        proxy_set_header X-Real-IP       $remote_addr;
        proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
        access_log on;
        access_log /usr/local/webserver/nginx/logs/www_access.log main;
    }

    location ~* \.php$
    {
        root /home/sam;
        try_files $uri =404;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi.conf;
        access_log on;
        access_log /usr/local/webserver/nginx/logs/www_phpproxy_access.log main;
    }
}
```

上面的配置表示nginx监听81端口，我自己把80端口分给了apache，当访问81端口下的/three目录下的静态文件时，nginx把请求转给80端口下的three文件，其实81端口下是没有任何文件的，nginx只是起到了转发请求的作用。location匹配php文件，当访问的是php文件，就把这个文件请求转到home/sam文件夹下，home/sam是apache的根目录，这样，nginx就把php请求转给了apache。最终，当访问http://120.27.104.77:81/three/index.php,nginx就把请求转给http://120.27.104.77:80/three/index.php，代理完成。

> 负载均衡

负载均衡指的是当一个服务器上的项目有很多用户时，服务器压力增大，需要其他服务器来帮忙分担请求，但是用户输入的网址不能变，所以，就应用到了nginx的负载均衡，负载均衡还是通过反向代理的方式，把请求转发给其他服务器，但是需要每一台服务器上的文件同步。下面是配置示例：

在http模块下添加负载均衡配置（默认使用的是轮询方式，weight表示是权重，下面表示82端口和80端口请求次数是1:1）：

```
upstream myserver {
    server 120.27.104.77:82 weight=1;
    server 120.27.104.77:80 weight=1;
}

server模块下添加location规则：

location ~*test.php {
    proxy_pass http://myserver;
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

80端口依然是apache，82端口是nginx服务器，当我访问81端口下的test.php，nginx就把请求转给82端口，再次刷新就把请求转给80端口，依次反复，这样就实现了负载均衡，相当于是有两台服务器在支撑。多设几个就有多个服务器支撑。


（完）