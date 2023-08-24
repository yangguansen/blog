---
title: typescript namespace嵌套引起的编译错误
date: 2023-08-24 18:11:52
tags:
	- JavaScript
---
记录一次遇到typescript使用namespace嵌套引起编译错误，以及如何解决
<!--more--> 

> 项目背景

项目使用gRPC作为前后端通信，所以会有后端生成的接口类型文件在前端项目中，其中很多类型结构长这个样子:

```
export namespace RuntimeStatus {
	export namespace Progress {
		export interface Quantity {
		  total?: number;
		  completed?: number;
		  unit?: string;
		}
		// export const a = '1';
	}

	export enum Severity {
		UNSPECIFIED = 0,
		NOTICE = 1,
		WARNING = 2,
		ERROR = 3,
		CRITICAL = 4,
	}
}
```

> 问题描述

在webpack@4 使用Babel编译时，它可以编译成功，但是由于项目决定升级到webpack@5,升级之后就报错：
```
Module parse failed: 'import' and 'export' may only appear at the top level (5:2)
File was processed with these loaders:
 * ./node_modules/@pmmmwh/react-refresh-webpack-plugin/loader/index.js
 * ./node_modules/babel-loader/lib/index.js
 * ./node_modules/source-map-loader/dist/cjs.js
You may need an additional loader to handle the result of these loaders.
| export let RuntimeStatus;
| (function (_RuntimeStatus) {
>   export let Progress;
|   let Severity = /*#__PURE__*/function (Severity) {
|     Severity[Severity["UNSPECIFIED"] = 0] = "UNSPECIFIED";

```

> 问题分析

我们可以看到报错显示`function`内部有一句`export xxx`, 这是报错的原因，于是我很纳闷，在怀疑是不是升级过程中插件版本不兼容导致的。

于是我重新用`create-react-app@5`生成一个新的项目，把这个代码片段插入进去，果然也是同样的报错信息，
但也同时实现了一个最小复现片段。

Typescript语法本身是支持这样的namespace嵌套的。
于是我就去Typescript官网的playground输入这段类型，
官网给出编译后的结果是：
```
var RuntimeStatus;
(function (RuntimeStatus) {
    let Severity;
    (function (Severity) {
        Severity[Severity["UNSPECIFIED"] = 0] = "UNSPECIFIED";
        Severity[Severity["NOTICE"] = 1] = "NOTICE";
        Severity[Severity["WARNING"] = 2] = "WARNING";
        Severity[Severity["ERROR"] = 3] = "ERROR";
        Severity[Severity["CRITICAL"] = 4] = "CRITICAL";
    })(Severity = RuntimeStatus.Severity || (RuntimeStatus.Severity = {}));
})(RuntimeStatus || (exports.RuntimeStatus = RuntimeStatus = {}));
```

官网编译成功且正确，于是我在typescript的github的issue中提了这个问题，他们回复这并不是typescript的问题，让我去问Babel问下，于是我又再Babel
的issue中提问。

作者很快答复了这确实Babel的插件`Babel-plugin-transform-typescript`编译的bug。

作者也给出了bug的原因：
在解析嵌套语句时，Babel会去解析嵌套语句的AST语法结构：
```
const transformed = handleNested(
  path,
  subNode.declaration,
  t.identifier(name),
);
if (transformed !== null) {
  isEmpty = false;
  const moduleName = subNode.declaration.id.name;
  if (names.has(moduleName)) {
    namespaceTopLevel[i] = transformed;
  } else {
    names.add(moduleName);
    namespaceTopLevel.splice(
      i++,
      1,
      getDeclaration(moduleName),
      transformed,
    );
  }
}
```
解析`subNode`的子节点，如果子节点只包含类型定义，比如`interface`, 也就是我们上面那段代码，那么解析结果就为`null`, 以上我们类型代码内部嵌套的`namespace Progress`并不包含值类型的定义，如果去掉注释中的`export const a = '1'`，`transformed`就不是`null`，
但对于解析结果为`null`的情况，需要删除这个AST节点，这样就不会有`export let Progress`这句了。Babel却忽略了这种情况。

>  解决问题

于是我加上了fix代码：
```
const transformed = handleNested(
  path,
  subNode.declaration,
  t.identifier(name),
);
if (transformed !== null) {
  ...
} else {
  namespaceTopLevel.splice(i, 1);
  i--;
}
```

并验证编译结果：
```
export let RuntimeStatus;
(function (_RuntimeStatus) {
  let Severity = /*#__PURE__*/function (Severity) {
    Severity[Severity["UNSPECIFIED"] = 0] = "UNSPECIFIED";
    Severity[Severity["NOTICE"] = 1] = "NOTICE";
    Severity[Severity["WARNING"] = 2] = "WARNING";
    Severity[Severity["ERROR"] = 3] = "ERROR";
    Severity[Severity["CRITICAL"] = 4] = "CRITICAL";
    return Severity;
  }({});
  _RuntimeStatus.Severity = Severity;
})(RuntimeStatus || (RuntimeStatus = {}));
```

可以看到`function`内部已经没有`export`了，说明对于`transformed`为`null`的AST并不会被`generated`。

我们再看下加上`const`语句的编译结果：
```
export let RuntimeStatus;
(function (_RuntimeStatus) {
  let Progress;
  (function (_Progress) {
    const a = _Progress.a = '1';
  })(Progress || (Progress = _RuntimeStatus.Progress || (_RuntimeStatus.Progress = {})));
  let Severity = /*#__PURE__*/function (Severity) {
    Severity[Severity["UNSPECIFIED"] = 0] = "UNSPECIFIED";
    Severity[Severity["NOTICE"] = 1] = "NOTICE";
    Severity[Severity["WARNING"] = 2] = "WARNING";
    Severity[Severity["ERROR"] = 3] = "ERROR";
    Severity[Severity["CRITICAL"] = 4] = "CRITICAL";
    return Severity;
  }({});
  _RuntimeStatus.Severity = Severity;
})(RuntimeStatus || (RuntimeStatus = {}));
```
可以看到`Progress`中只有const语句被generated了，而不包含`interface Quantity`。

> 结果

提交PR, contributor review and merge.
