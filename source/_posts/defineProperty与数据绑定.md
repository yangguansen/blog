---
title: defineProperty与数据绑定
date: 2017-03-21 15:59:47
tags:
	- JavaScript
---
defineProperty用来声明一个对象，与对象字面量不同的是，它可以声明内部值get(设置)和set(获取)时的方法，如：
```
var obj = {};		
Object.defineProperty(obj, 'hello', {
	get: function() {
		console.log('get!');
	},
	set: function(value) {
		console.log('set!');
	}
});
obj.hello = 1;	//打印set
obj.hello;	//打印get
```
由此，我们可以利用此特性实现一个简单的双向绑定：
```
var obj = {};		
Object.defineProperty(obj, 'hello', {
	get: function() {
		console.log('get!');
	},
	set: function(value) {
		document.querySelector(".a").value = value;
		document.querySelector(".b").innerHTML = value;
	}
});
document.addEventListener('keyup', function(e){
	obj.hello = e.target.value;
})
```
###documentFragment
先来看一段代码：
```html
<div id="app">
	<input type="text" class="a">
	<span class="b"></span>
</div>
```
```
function nodeToFragment(node)
{
	var flag = document.createDocumentFragment();
	while (node.firstChild){
		flag.append(node.firstChild);	//append方法会把节点从A移动到B
	}
	return flag;
}
var dom = nodeToFragment(document.getElementById("app"));
console.log(dom);
```
这里打印会把div标签里的所有标签打印出来，而html中div的标签内为空，是因为这段代码把div里需要操作的标签全都转到了documentFragment里了，因为在documentFragment里操作标签比在dom中操作要性能好很多。

###简单的通过实例进行数据绑定：
```
function compile(node, vm){		//解析赋值
	var reg = /\{\{(.*)\}\}/;
	if(node.nodeType === 1){
		var attr = node.attributes;
		for(var i = 0; i < attr.length; i++){
			if(attr[i].nodeName == "v-model"){
				var name = attr[i].nodeValue;
				node.value = vm.data[name];
				node.removeAttribute('v-model');
			}
		}
	}
	if(node.nodeType === 3){
		if(reg.test(node.nodeValue)){
			var name = RegExp.$1;
			name = name.trim();
			node.nodeValue = vm.data[name];
		}
	}
}	
function nodeToFragment(node, vm)	//把节点转移到documentFragment进行操作
{
	var flag = document.createDocumentFragment();
	while (node.firstChild){
		compile(node.firstChild,vm);
		flag.append(node.firstChild);	//append方法会把节点从A移动到B
	}
	return flag;
}
function Vue(options)
{
	this.data = options.data;
	var id = options.el;
	var dom = nodeToFragment(document.getElementById(id), this);
	console.log(dom)
	document.getElementById(id).appendChild(dom);
}
var vm = new Vue({
	el:'app',
	data:{
		text:'hello World'
	}
})
```
接下来再利用defineProperty进行set数据绑定，即可实现动态双向绑定：
修改代码：
```
function compile(node, vm){		//解析赋值
	var reg = /\{\{(.*)\}\}/;
	if(node.nodeType === 1){
		var attr = node.attributes;
		for(var i = 0; i < attr.length; i++){
			if(attr[i].nodeName == "v-model"){
				var name = attr[i].nodeValue;
				node.addEventListener('input', function(e){
					vm.data[name] = e.target.value;
				})
				node.value = vm.data[name];
				node.removeAttribute('v-model');
			}
		}
	}
	if(node.nodeType === 3){
		if(reg.test(node.nodeValue)){
			var name = RegExp.$1;
			name = name.trim();
			node.nodeValue = vm.data[name];
		}
	}
}	
function nodeToFragment(node, vm)	//把节点转移到documentFragment进行操作
{
	var flag = document.createDocumentFragment();
	while (node.firstChild){
		compile(node.firstChild,vm);
		flag.append(node.firstChild);	//append方法会把节点从A移动到B
	}
	return flag;
}
function Vue(options)
{
	this.data = options.data;
	var data = this.data;
	observe(data, this);
	var id = options.el;
	var dom = nodeToFragment(document.getElementById(id), this);
	console.log(dom)
	document.getElementById(id).appendChild(dom);
}
function defineReactive(obj, key, val)
{
	Object.defineProperty(obj, key, {
		get:function(){
			return val
		},
		set:function(newVal){
			if(newVal === val) return;
			val = newVal;
			console.log(val);
		}
	})
}
function observe(obj, vm)
{
	Object.keys(obj).forEach(function(key){
		defineReactive(vm.data, key, vm.data[key]);
	})
}
var vm = new Vue({
	el:'app',
	data:{
		text:'hello World'
	}
})
```
以上便可实现在输入时，同时打印出data.text，已经和vm.data.text实现了绑定