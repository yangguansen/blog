---
title: react项目搭建流程
date: 2018-08-01 13:51:30
tags:
    - JavaScript
---

基于create-react-app,记录react项目搭建流程。
<!--more-->

## ant-design与css
- css-modules的css模块化和ant-design的样式冲突。
解决办法：把webpack-dev中的css解析规则和node_modules分开，原先的规则加入`include: /node_modules|antd\.css/`，然后把css规则复制一份，再排除antd.css：`exclude: /node_modules|antd\.css/`。
并定义css命名规则：` modules: true, localIdentName: "[local]_[hash:base64:5]"`。
需要支持less的话，就在use中push一条：`{loader:require.resolve('less-loader')}`。
ant-design懒加载：在babel的options中添加属性：
`plugins: [
    ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": 'css' }]
    ]`

## react-router
- 引入`react-router-dom`
- 编写router配置：
```
const routes = [
    {
        path: '/home/dashboard',
        component: DashBoard
    },
    {
        path: '/home/sip/collect',
        component: Collect
    },
    {
        path: '/home/sip/defectWrite',
        component: DefectWrite
    }]
```

- 然后jsx写入到html中:

```
{routes.map( ( route, index ) => (
    <Route key={index} path={route.path} component={route.component}/>
    ) )}
```

- js跳转路由：

```
    this.props.history.push( key );
```

## mobx

- 声明store
```
import {observable, action, computed} from 'mobx';

class State {
    @observable a = 0;  // 声明变量
    
    @computed
    get b() {   // 依赖其他变量的变量
        return this.a+3;
    }
}

class Actions {
    constructor({state}){
        this.state = state; // 将state初始化
    }

    @action //action改变变量
    add = () =>{
        this.state.a++;
    }
}

const state = new State();

export default new Actions({state});
```

-   业务层与数据层的连接

```
@inject( 'state', 'actions' )   //将store注入到业务层
@observer
class DashBoard extends React.Component {
    render() {
        const {state, actions} = this.props;
        <p>a:{state.a}</p>
        <Button type="primary" onClick={actions.add}>+</Button>
    }
}
```

## 插槽

- 创建插槽模板
```
import React from 'react';
import style from './style.less';

class Card extends React.Component{
    render(){
        return (
            <div className={style.card}>
                <div className={style.cardTitle}>{this.props.title}</div>
                {this.props.children}
            </div>
        )
    }
}

export default Card;
```

- 使用插槽
```
import Card from '@/components/common/Card';

<Card className={style.collectInfo} title='基本信息'>
    <UploadPhoto></UploadPhoto>
    <Message formData={this.state.collectMessage}></Message>
</Card>
```
