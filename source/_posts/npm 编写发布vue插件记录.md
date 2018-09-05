---
title: npm 编写发布vue插件记录
date: 2018-01-21 21:42:45
tags:
	- 总结
---

> 项目背景

本文记录自己学习npm编写vue插件，并导出包文件，发布到npm平台，供其他地方全局调用。

> 过程记录

首先使用vue-cli脚手架下载一个简版的vue框架`webpack-simple`,然后在src文件中新建lib文件夹用来存放插件代码，在lib中新建index.js和index.vue，在index.vue中写一个按钮，表示是我们的插件

index.vue代码：
```
<template>
  <div class="button">
    <button @click="handler">这是一个按钮</button>
  </div>
</template>

<script type="application/ecmascript">
    export default {
      name: 'test-button',
      data() {
        return {};
      },
      methods: {
        handler() {
          console.log( '点击按钮' );
        }
      }
    };
</script>

<style type="text/less" lang="less" scoped>
  .button {
    width: 100px;
    height: 20px;
    text-align: center;
    margin: 0 auto;
  }
</style>

```

在index.js中将组件模块化，并导出组件

index.js代码：
```
import Index from './index.vue';

const button = {

  //通过vue提供的install方法来将组件注册到vue全局中。
  install( Vue ) {
    Vue.component( Index.name, Index );
  }
}

if ( typeof window !== 'undefined' && window.Vue ) {
  window.Vue.use( Index );
}

//  将组件导出
export default button;

```

> 打包发布

插件已经编写完毕，我们只需要访问index.js文件就能使用该组件，接下来就是通过webpack将组件代码打包，在项目根目录新建一个webpack-library.js文件来编写webpack打包组件的配置，里面的配置跟webpack.config.js大致相同，但是需要修改文件打包入口出口。

```
//  webpack.config.js的入口是main.js文件，表示是整个项目的入口，但是我们这里只是要打包按钮组件，所以入口要改为lib下的index.js文件
entry: './src/lib/index.js',
output: {
path: path.resolve(__dirname, './dist'),
publicPath: '/dist/',
//  为了将组件打包出来的文件区分出来，我们要重命名打包后的js文件名
filename: 'test-button.js',
//  插件的名字
library: "TestButton",
//  不懂，反正写'umd'就是了
libraryTarget: "umd",
umdNamedDefine: true
},

```

接下来是package.json的配置：
声明一个包文件入口，表示别人下载了组件后，使用时的入口文件，其实就是上面配置后，生成的文件地址

```
"main": "./dist/test-button.js",
```
新建一个npm命令用来打包组件

```
"scripts": {
    ···
    "publish": "webpack --config webpack-library.js"
    ···
  },
```

然后运行`npm run publish`开始打包，再运行`npm publish`发布npm包，发布成功后，我们就可以在npm上看到自己写的包了，使用时，就运行`npm install vue-plugin`,然后在vue项目中引入，注册：

```
import TestButton from 'vue-plugin';
Vue.use( TestButton );
```

使用：
```
<test-button></test-button>
```

在项目中就可以看到自己编写的组件了。

(完)