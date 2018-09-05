---
title: 今天遇到的i++问题之记录
date: 2015-12-02 17:38:52
tags:
	- JavaScript
---
今天逛贴吧看到的，与自己预想的不同，于是在群里求解后方得知答案，遂记录之。代码来袭。
```
function a(){
    var i=1;
    i++;
    alert(i); //2
}
var c = a();
c();
```

```
function a(){
    var i=1;
    alert(i++); //1
}
var c = a();
c();
```
之所以是1是因为alert(i++)这句的过程是先取后加，取得是加之前的值。
```
function a(){
    var i=1;
    alert(i++); //1
    alert(i);   //2	加了之后就是2了
}
var c = a();
c();
```
所以alert(i++)这句话的过程是这样的   第一步：继承自上一句i=1 。  第二步：弹！  这时弹得值为1    第三步：i=i+1   这时的i就是2了。    所以下面那一句的alert(i)也就是2了