---
title: 如何编写一个webpack-loader
date: 2018-12-13 14:14:52
tags:
	- JavaScript
---
loader在webpack中是一个很重要的角色，babel-loader,vue-loader,json-loader等等，都给我们的开发带来了很好的体验，本篇博文就尝试讲解如何编写一个loader。
<!--more--> 

> 目标

写一个txt-loader，可以引入txt文本，并在代码中使用。

> 开始

首先`mkdir loader`, `cd loader && npm init`来初始化一个前端工程化项目,然后在项目根目录下新建个我们需要用到的`test.txt`,在里面随便写点内容`abcde`。然后新建index.js作为入口文件，并在其中引入txt文件：
index.js:
```
const txt = require('./test.txt');
console.log(`11, ${txt}`);
```

然后安装webpack,并简单配置一下webpack的入口，出口，loader:
```
var path = require( 'path' );
var CleanWebpackPlugin = require( 'clean-webpack-plugin' );
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    mode: 'production',
    entry: './index.js',

    output: {
        path: path.resolve( __dirname, './dist' ),
        filename: 'bundle.[hash:8].js',
    },
    module: {
        rules: [
            {
                test: /\.txt$/,
                use: {
                    loader: path.resolve( __dirname, './loaders/txt-loader/index.js' )
                }

            }
        ]
    },

    plugins: [
        new CleanWebpackPlugin( './dist' ),

        new HtmlWebpackPlugin( {
            title: 'Output Management'
        } )
    ],
}
```
对于txt的loader, webpack默认是从node_modules中去查找对应的loader，想要引入本地loader时，有两种方法：

 1. 如以上代码，只需要写明本地路径就可以
 2. 在config中配置resolveLoader属性,这样webpack就会依次从路径中查找：
```
resolveLoader: {
    modules: [
      'node_modules',
      path.resolve(__dirname, 'loaders')
    ]
  }
```

当你自己写的loader在本地测试通过后，就可以`npm publish`发布在npm平台上，供他人使用了。

那我们就在本地新建txt-loader文件夹，并在里面新建index.js来编写核心代码：
```
module.exports = function (content) {
    var result = content.split('').reverse().join('').toUpperCase();
    return `module.exports = '${result}'`;
};
```
这个loader的功能会把字母倒叙并转换成大写，这样我们的loader的功能就编写完了。
> 注意:对于一个txt格式使用多个loader时，loader的顺序是倒序的.
```
use: [
  'bar-loader',
  'foo-loader'
]
```
先经过foo-loader，再经过bar-loader。
所以每一个loader方法都会接收上一个loader的产出，因此loader的产出应该为：
```
return `module.exports = '${result}'`;
```
具体的介绍详见[webpack官方介绍](https://webpack.docschina.org/contribute/writing-a-loader/),这里就不墨迹了。

最后，loader的编写就完成了，检测下成果：将打包的index.js引入到html中，打开html，打印出了EDCBA。
完美！

（完）