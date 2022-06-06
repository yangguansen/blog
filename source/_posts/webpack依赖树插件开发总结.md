---
title: webpack依赖树插件开发总结
date: 2022-03-17 17:22:14
tags:
    - Webpack
---

记录webpack依赖分析插件开发的过程
<!--more-->

事情的起因是某日突然接到需求，是要对一个从外部引入的项目进行改造，需要快速熟悉项目，但是当打开这个项目代码时，发现外部插件非常多，全局公共组件多达600+，编译一次要2分钟，这种项目看代码就比较恼火，于是心想是否能有一个插件可以概览整个项目中每个文件中引入的依赖，网上搜索一番，并无此类插件，于是心想不如自己造一个。

查阅webpack插件文档及相关文章，plugin hooks中的import hook，可以在执行import语句时，触发事件。而项目中刚好是通过import引入依赖，于是试试写点代码，看看会发生什么
```
apply(compiler) { 
	compiler.hooks.compilation.tap(this.pluginName, (compilation, { normalModuleFactory }) => {   
   const handler = (parser, options) => {
        parser.hooks.import.tap(this.pluginName, (statement, source) => {
        debugger;
		//看看参数能拿到什么有用信息          
      });
    }
    normalModuleFactory.hooks.parser
      .for("javascript/auto")
      .tap(this.pluginName, handler);
  });
}
```

那webpack插件开发过程中怎么debug呢？总不能老是console吧。
查阅一番，使用`node --inspect-brk index.js`即可以在浏览器中进行debug, index.js是项目的入口文件，我初始化一个react项目，通常我们`npm run dev`启动项目是封装了node命令，那这里我们就要改变启动命令为：`cross-env NODE_ENV='development' node --inspect-brk client/scripts/start.js`
然后在Google浏览器的中输入`chrome://inspect/`,
![enter image description here](http://jxqdh.91sam.com/img/1647425181-1872-6231b69d2dbaa-566462.png)
点击这个inspect按钮就可以调试啦。
那我们拿到的hook参数是：
![enter image description here](http://jxqdh.91sam.com/img/webpack-2.png)
![enter image description here](http://jxqdh.91sam.com/img/webpack-3.png)
可以看到source就是import需要引入的文件名，statement是这个语句的AST。
但是怎么知道webpack当前是在处理哪个文件呢？
答案是在parser，parser表示处理当前文件的解析器，webpack对不同文件就会使用不同的parser,可以看到webpack处理js文件是通过`javascript/auto`的type来处理。
![enter image description here](http://jxqdh.91sam.com/img/webpack-4.png)
parser.state.current就是webpack当前正在处理的文件。

假如我们的代码是这样写的：

index.js
```
import 'react-app-polyfill/ie11';
import 'react-app-polyfill/stable';
import ReactDOM from 'react-dom';
import React from 'react';
import { Provider } from 'react-redux';
import store from '@src/store/index';
import App from './app';
```
那我们的import hook就会触发7次，这样我们就拿到了这个文件的依赖：
```
{
	resource: ${绝对路径}/index.js
	deps: [
		'react-app-polyfill/ie11',
		'react-app-polyfill/stable',
		'react-dom',
		'react',
		'react-redux',
		'@src/store/index',
		'./app'
	]
}
```
在第一个文件处理完后，接着就会在依赖中的继续触发import hook，但是我们的需求并不需要知道第三方依赖的import, 所以就过滤掉`node_modules`下的文件
```
 if (parser.state.current.resource.includes('node_modules') || source.includes('node_modules')) {
       return;
 }
```
接下来会处理的将是`@src/store/index`和`./app`的依赖，以此类推,直到处理完所有的文件，我们拿到了所有的文件及其依赖。
![enter image description here](http://jxqdh.91sam.com/img/webpack-5.png)

那么怎么知道处理完成所有文件呢？
我们选用`finishModules` hook
```
compiler.hooks.make.tap(this.pluginName, (compilation) => {
  compilation.hooks.finishModules.tap(this.pluginName, (modules) => {
    // 所有modules处理完成
  });
});
```
接下来就是要把一维数组转成树结构，由于我们监听的是import hook, 所以第一个触发这个hook的文件一定是入口文件，即数组的第一项，那就从第一个文件的依赖中来递归组织树结构
```
function getDependencies(file, array) {
  const item = array.find(v => v.rawRequest === file);
  return item?.dependencies || [];
}
function walk(dependencies, array) {
  dependencies.forEach(v => {
    v.name = v.source;
	 //	只处理我们自己写的代码
    if (v.source.slice(0, 1) === '@' || v.source.slice(0, 1) === '.') {
      v.children = getDependencies(v.source, array);
      walk(v.children, array);
    }
  });
}

const transformArrayToTree = (array) => {
  if (!array || array?.length === 0) return null;
  const tree = {};
  //  默认第一个触发import钩子的文件是入口，作为树的根节点
  tree.name = array[0].rawRequest;
  tree.children = array[0].dependencies;
  walk(tree.children, array);

  return tree;
};

```
这样便能拿到树结构了
![enter image description here](http://jxqdh.91sam.com/img/webpack-6.png)

然后就是要把数据转成可视化页面，这里我们需要启动一个node web服务，渲染一个模板页面，把数据填充进去，并自动打开浏览器页面
```

function renderViewer(jsonString) {
  return new Promise((resolve) => {
    fs.readFile(path.resolve(__dirname, './dependencies.html'), 'utf-8', (err, data) => {
      if (err) throw err;
      const html = data.replace(/<%=(\w+)%>/g, (match, $1) => jsonString);
      resolve(html);
    });
  });
}

function openBrowser(url, info) {
  try {
    opener(url);
    console.log(info);
  } catch (err) {
    console.error(`Opener failed to open "${url}":\n${err}`);
  }
}

async function startServer(jsonString) {
  const port = 8888;
  const host = '127.0.0.1';
  const isOpenBrowser = true;
  const html = await renderViewer(jsonString);
  http.createServer((req, res) => {
    if (req.method === 'GET' &amp;&amp; req.url === '/') {
      res.writeHead(200, { 'Content-Type': 'text/html' });
      res.end(html);
    } else {
      res.end('blank page');
    }
  }).listen(port, host, () => {
    const url = `http://${host}:${port}`;

    const logInfo = (
      `Webpack Source Code Dependencies Analyzer is started at ${(url)}\n`
      + `Use ${('Ctrl+C')} to close it`
    );

    if (isOpenBrowser) {
      openBrowser(url, logInfo);
    }
  });
}

```

打开的页面是这样子：
![enter image description here](http://jxqdh.91sam.com/img/webpack-7.png)

通过点击节点来查看每个页面的依赖。

但是，当检查这些依赖时，发现通过`import()`动态加载的组件并没有被收集进来，查看webpack hook文档说是import hook会触发所有import，这里就很疑惑。几经搜索与思考，webpack只是打包工具，真正处理这些文件的，还是对应的loader, 那处理import的解析器是babel, babel中识别import是ImportDeclaration，而import()是ImportExpression, 也就是说import()并不会触发import hook, 于是又去查看webpack Parser的源码，发现有个importCall, importCall却并没有出现在webpack文档中，于是试探性的添加一个importCall hook， 果不其然，在解析import()语句时，触发了importCall hook。

至此，一个依赖分析插件的雏形就已经出来了，后续就是一些细节上的优化，比如同个文件被两个文件以不同命名的方式引入，这种情况还需要处理下。

（完）

github源码: [链接地址] (https://github.com/yangguansen/webpack-code-dependency-analysis-plugin)
