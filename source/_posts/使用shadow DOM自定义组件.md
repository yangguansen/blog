---
title: 使用shadow DOM自定义组件
date: 2022-05-24 12:44:30
tags: 
    - JavaScript
---

在这篇文章中，我们主要介绍Shadow DOM。
<!--more-->

Shadow DOM被设计为基于组件创建应用的工具，对比其他普通DOM，它提供了：

- <b>隔离性</b>：在shadow DOM中元素是与普通DOM隔离的，我们使用`document.querySelector()`将不会返回node节点，同样也不需要担心class/id冲突。

- <b>CSS作用域</b>：在shadow DOM中的CSS只在其内部生效，不会影响页面上的其他样式。

- <b>组装</b>：可以设计一个可声明的，基于标记的组件。

与此同时，Shadow DOM对比普通的DOM还有以下两点不同：
- 在页面上如何创建和使用DOM。
- 在页面上其他地方它是如何工作的。

通常情况下，你创建DOM节点，然后把它作为子元素添加到另一个元素中， 在shadow DOM中，你创建一个有作用域的DOM树，然后添加到其他子元素中，这个DMO树就称为`shadow tree`。包含这个tree的元素就称为`shadow host`。你添加到shadow tree中的任何元素，都是在host中的，包括`<style>`标签，这就是shadow DOM如何实现CSS样式作用域的。

#### 创建Shadow DOM

`shadow root`是document片段的根节点，当创建shadow DOM时，你就得到了一个shadow root，为一个元素创建shadow DOM,使用`element.arrachShadow()`:

```
var header = document.createElement('header');
var shadowRoot = header.attachShadow({mode: 'open'});
var paragraphElement = document.createElement('p');

paragraphElement.innerText = 'Shadow DOM';
shadowRoot.appendChild(paragraphElement);
```

#### Shadow DOM中的组装

组装是Shadow DOM中非常重要的一个特性。
当写HTML时，你通过拼接不同的元素比如`<div>` `<header>` `<form>`等来实现页面UI，组装成的web应用，这些标签可以相互搭配。

组装定义了为什么`<select><form><video>`等标签是可扩展的，并且可以接收其他HTML元素作为子元素实现一些特别的功能。

比如，`<select>`知道在设置了预选项的下拉框组件中，如何渲染`<option>`元素。

Shadow DOM采用以下特性用来实现组装。

#### 轻量DOM

这是一个自定义组件的写法，自定义标签中包含真实DOM，想象一下如果你想创建一个组件叫`extended-button`, 这个组件扩展了原生HTML按钮，并且你想包含一个icon和一些文字，写法大概如下：

```
<extended-button>
  <!-- image 和 span 是 extended-button 的轻量 DOM -->
  <img src="boot.png" slot="image">
  <span>Launch</span>
</extended-button>
```

这个“extended-button”是你定义的自定义组件，其内部的HTML称为轻量DOM。Shadow DOM在这里就是你创建的“extended-button”，它定义了它的内部结构，CSS作用域，并封装了内部实现。

#### Templates

当你在web页面上必须重复使用相同的DOM结构时，最好是定义一些模板来实现复用，这在以前也是可能实现的，但是现在可以使用HTML <template>标签更容易实现，这个元素及其内容在DOM中不会被渲染，但是可以在Javascript中被引用。

看个例子：

```
<template id="my-paragraph">
  <p> Paragraph content. </p>
</template>
```

这不会在页面上显示，除非你在Javascript中引用它，并把它添加到其他DOM中。

```
var template = document.getElementById('my-paragraph');
var templateContent = template.content;
document.body.appendChild(templateContent);
```

这在一些框架中可以实现，但是在之前提到的，它更加原生，并且有不错的浏览器兼容性：
![s2 (1).png](https://km.woa.com/gkm/api/img/cos-file-url?url=https%3A%2F%2Fkm-pro-1258638997.cos.ap-guangzhou.myqcloud.com%2Ffiles%2Fphotos%2Fpictures%2F202205%2F1653366684-4798-628c5f9c752ad-624325.png&is_redirect=1)

Templates可以和自定义元素搭配，起到更好的效果，我们将自定义元素，此时你应该知道浏览器的`customElements`API，允许你自定义标签来渲染。

让我们用templates的内容作为shadow DOM来定义个web component，我们称为`my-parrgraph`：

```
customElements.define('my-paragraph',
 class extends HTMLElement {
   constructor() {
     super();

     let template = document.getElementById('my-paragraph');
     let templateContent = template.content;
     const shadowRoot = this.attachShadow({mode: 'open'}).appendChild(templateContent.cloneNode(true));
  }
});
```

这里的关键点是我们添加了一个template子元素的副本到shadow root中，副本是通过`Node.cloneNode()`方法创建的。

同样我们可以添加一些样式到shadow DOM中，我们可以在template中添加`<style>`标签，它在普通DOM中不会生效。

例如：我们可以修改我们的template：

```
<template id="my-paragraph">
  <style>
    p {
      color: white;
      background-color: #666;
      padding: 5px;
    }
  </style>
  <p>Paragraph content. </p>
</template>
```

现在我们的自定义元素可以这样使用：
`<my-paragraph></my-paragraph>`

#### Slots

Templates有一些缺点，其中之一是它的静态内容不允许我们渲染我们的动态数据。

这时，`<slot>`就派上用场了。

你可以认为slots是作为占位符，允许你把你自己的HTML放到tempalte中，它允许你创建通用的HTML templates并通过添加slots实现自定义。

让我们看template和slot如何搭配：

```
<template id="my-paragraph">
  <p> 
    <slot name="my-text">Default text</slot> 
  </p>
</template>
```

如果元素添加自定义标签时，slot的内容没有定义，或者浏览器不支持slots,`<my-paragraph>`只会显示它的内容“Default text”。

为了定义slot的内容，我们应该在`<my-paragraph>`中包含一个元素，这个元素包含slot属性，并且它的slot值等于我们想要填充的slot的name。

比如：

```
<my-paragraph>
 <span slot="my-text">Let's have some different text!</span>
</my-paragraph>
```

这个例子是我们插入了一个span标签，他有slot属性，slot的值等于my-text的name.

在被浏览器渲染之后，将会显示为以下DOM树：

```
<my-paragraph>
  #shadow-root
  <p>
    <slot name="my-text">
      <span slot="my-text">Let's have some different text!</span>
    </slot>
  </p>
</my-paragraph>
```

`#shadow-root`只是一个shadow DOM存在的指示符

#### 样式
shadow DOM的组件样式可以定义在主页面中，也可以自定义在shadow DOM中。

#### 自定义组件样式

<b>CSS作用域</b>是Shadow DOM的最棒的特性之一。

- 外部的CSS选择器无法选中你的组件。
- 组件自定义样式不会影响页面其他部位，他们只在host元素中生效，所以不必担心id命名冲突。

来看个例子：
```
#shadow-root
<style>
  #container {
    background: white;
  }
  #container-items {
    display: inline-flex;
  }
</style>

<div id="container"></div>
<div id="container-items"></div>
```

所有的样式只会在#shadow-root中生效。

你也可以使用`<link>`元素来引用样式，也只会在#shadow-root中生效。

#### :host伪类
`:host`允许你选中shadow tree的根节点。
```
<style>
  :host {
    display: block; /* 默认自定义元素是 display: inline */
  }
</style>
```

有一个需要注意点的是外层的:host的优先级高于元素内部的:host，并且:host只在shadow root的作用域中生效。
`:host(选择器)`允许你在符合状态时选中host，因此你可以在响应交互状态时使用它。
```
<style>
  :host {
    opacity: 0.4;
  }
  
  :host(:hover) {
    opacity: 1;
  }
  
  :host([disabled]) { /* style when host has disabled attribute. */
    background: grey;
    pointer-events: none;
    opacity: 0.4;
  }
  
  :host(.pink) > #tabs {
    color: pink; /* color internal #tabs node when host has class="pink". */
  }
</style>
```

#### 使用:host-context伪类匹配主题
`:host-context(选择器)`伪类能够在匹配（选择器）时，选中元素，通常情况下用来结合主题使用。比如：
```
<body class="lightheme">
  <custom-container>
  …
  </custom-container>
</body>
```
`:host-context(.lightheme)`能够生效，并渲染样式。
```
:host-context(.lightheme) {
  color: black;
  background: white;
}
```

#### 从组件外侧添加样式
你可以从组件的外侧给组件添加样式，比如这样：
```
custom-container {
  color: red;
}
```

<b>外层的样式比shadow DOM中的样式具有更高的优先级。</b>

例如，我们外层给的样式：
```
custom-container {
  width: 500px;
}
```

可以覆盖组件的样式规则：
```
:host {
  width: 300px;
}
```

#### 事件模型
Shadow DOM的事件冒泡机制比较特别，事件的target都由Shadow DOM封装，即组件内部元素的事件，target都会是Shadow DOM组件本身。
以下是Shadow DOM的冒泡事件列表：
- Focus Events: blur, focus, focusin, focusout
- Mouse Events: click, dblclick, mousedown, mouseenter, mousemove, etc.
- Wheel Events: wheel
- Input Events: beforeinput, input
- Keyboard Events: keydown, keyup
- Composition Events: compositionstart, compositionupdate, compositionend
- DragEvent: dragstart, drag, dragend, drop, etc.

#### 自定义事件
自定义事件默认不会冒泡到Shadow DOM外层，如果你想捕获自定义事件且想让它冒泡，你需要添加`bubbles: true`和`composed: true`作为选项。代码如下：
```
var container = this.shadowRoot.querySelector('#container');
container.dispatchEvent(new Event('containerchanged', {bubbles: true, composed: true}));
```

#### 浏览器支持
为了检查浏览器是否支持Shadow DOM功能，可以像这样：
```
const supportsShadowDOM = !!HTMLElement.prototype.attachShadow;

```
通常情况下，Shadow DOM与普通DOM有一些行为上的差异，我们可以使用SessionStack库来收集用户事件，网络数据，异常捕获，debug信息，把他们发送到服务端，来让我们能够复现问题。

#### 案例
Youtube里大量使用了自定义组件：
![enter image description here](https://km.woa.com/gkm/api/img/cos-file-url?url=https%3A%2F%2Fkm-pro-1258638997.cos.ap-guangzhou.myqcloud.com%2Ffiles%2Fphotos%2Fpictures%2F202205%2F1653367203-0293-628c61a3072f1-872116.png&is_redirect=1)
其是使用了Polymer library来构建自定义组件。
后面会输出一篇Polymer library的学习笔记，敬请期待。

参考文献：

- https://blog.sessionstack.com/how-javascript-works-the-internals-of-shadow-dom-how-to-build-self-contained-components-244331c4de6e 
