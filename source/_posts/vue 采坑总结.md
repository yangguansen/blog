---
title: vue 采坑总结
date: 2017-12-31 16:33:45
tags:
	- 总结
---

> 项目背景

本文记录自己学习使用vue中所踩到的坑。

> 采坑记录

1、postcss配置
在项目中，如果是用rem单位来布局的话，css中最长用到的就是autoprefixer和px2rem，这两个都是postcss的插件，那么如何在webpack中配置postcss呢？就连webpack上的文档经过亲手尝试，也是报错，经过多重搜索，最后是配置这样的,在module中的rules选项中配置，因为是在解析vue文件时解析css,所以要配在匹配vue的文件下

```
{
    test: /\.vue$/,
    loader: 'vue-loader',
    options: {
      postcss: [
        require( 'autoprefixer' )( {
          browsers: [ 'last 10 versions', 'Firefox >= 20', '> 1%', 'iOS 4', 'android >= 2.0', 'and_uc > 1' ]
        } ),
        require( 'postcss-plugin-px2rem' )( {
          rootValue: 10,
          minPixelValue: 2,
        } )
      ]
    }
  },
```

然后再加上自适应的方法，动态判断屏幕大小来改变字体大小，如果设计稿宽度是375px：
```
var response = function () {
	var w = document.documentElement.clientWidth;
	document.documentElement.style.fontSize = w / 37.5 + 'px';
};
window.onresize = function () {
	response();
	clearTimeout( this.responseTimer );
	this.responseTimer = setTimeout( response, 300 );
};
```


2、webpack代码分割
webpack提供了代码分割功能，可以根据路由来动态加载所需模块，减小了初次加载的体积，在编写路由模块时，将路由component动态引入：
```
const Index = ( r ) => require.ensure( [], () => r( require( './index.vue' ) ) , 'moduleA');
import ChildrenA from './childrenA/router';

export default {
  path: '/a',
  name: 'module-a',
  component: Index,
  children: [ ChildrenA ]
}
```

第一行代码中，`moduleA`表示为该component命名，当webpack打包时，会将所有同名的组件打包到一个js文件中，当加载到`/a`的路由时，就会加载该js文件，这个文件中包含了所有名字为`moduleA`的模块。

3、vuex局部模块
vuex可以理解为vue中专门用来处理事务数据的，里面变量在全局可访问，当项目变大，变量变多时，vuex提供了modules属性，可以将数据以模块来划分。
```
import moduleA from '../moduleA/store';
import moduleB from '../moduleB/store';

const modules = {
  moduleA,
  moduleB
};

export default new Vuex.Store({
  modules
})
```

`moduleA`为子模块，它又可以在内部继续划分为多个子模块:

```
export default {
  namespaced: true,
  modules: {
    children: childrena
  },
  state,
  types,
  mutations,
  actions
}
```

需要注意的是，当在内部划分成子模块时，要加上namespaced属性，这样子模块就可以继承父模块的命名空间，当在组件中调用moduleA下的子模块的action时，就要加上前缀:
```
...mapActions( {
    'setmoduleA': 'moduleA/moduleAfn',
    'setmoduleB': 'moduleB/children/moduleBfn'
  } ),
```

4、webpack proxy代理
在devServer中添加proxy代理，要添加`pathRewrite`属性，这样，当我们请求`/api/v2/api/Community/Navigations`时，就会转到`http://www.baidu.com/v2/api/Community/Navigations`
```
proxy: {
  '/api/': {
    target: 'http://www.baidu.com/',
    changeOrigin: true,
    pathRewrite: {
      '^/api': ''
    }
  }
}
```

5、webpack压缩问题
webpack自带的压缩功能uglifyjsplugin方法无法对ES6的语法进行压缩，要先在module中添加bable-loader,babel会把ES6转为ES5的语法，然后就可以压缩代码了。
```
module:{
  rules: [
    {
      test: /\.js$/,
      exclude: /node_modules/,
      use: {
        loader: 'babel-loader',
      }
    }
  ],
},
plugins: [
  new webpack.optimize.UglifyJsPlugin(),
]
```
（未完待续）