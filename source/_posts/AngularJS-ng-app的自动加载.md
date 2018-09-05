---
title: AngularJS ng-app的自动加载
date: 2015-09-30 18:12:02
tags:
	- AngularJS
---
今天练习AngularJS遇到了这个问题，ng-app=“值”，便无法实现，遂百度找到了答案，共享之。。

ng-app是angular的一个指令，代表一个angular应用（也叫模块）。使用ng-app或ng-app=""来标记一个DOM结点，让框架会自动加载。也就是说，ng-app是可以带属性值的。如果想要实现自动加载，那么就不能让ng-app带有属性值。
```
<html>
        <body ng-app>
                <div>div1:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>
```
1、不含ng-app，无法自动加载，这个比较好理解。
```
<html>
        <body>
                <div>div1:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>```
2、含有2个ng-app，那么只会自动加载第一个，这个也好理解。
```
<html>
        <body>
                <div ng-app>div1:{{1+3*2}}</div>
                <div ng-app>div2:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>```
3、ng-app带有属性，不能自动加载
```
<html>
        <body>
                <div ng-app="app1">div1:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>```
4、不带属性的在前，带属性的在后。ng-app标记的模块可以自动加载
```
<html>
        <body>
                <div ng-app>div1:{{1+3*2}}</div>
                <div ng-app="app1">div1:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>```
5、带属性的在前，不带属性的在后。ng-app标记的模块不能自动加载
```
<html>
        <body>
                <div ng-app="app1">div1:{{1+3*2}}</div>
                <div ng-app>div1:{{1+3*2}}</div>
                <script src="angular.js"></script>
        </body>
</html>```