# react脚手架中的组件

## 项目下src目录的组件常见结构
### 结构一：使用组件时引入具体的组件文件（优点是清晰明了，缺点是引入组件需要写具体的组件文件）
~~~
src                       // 主要源码文件
├── components            // 组件目录，存放多个不同的组件
│   └── Hello             // 具体的组件目录，存放对应组件的js、jsx、css代码
│       ├── Hello.jsx     // 组件，可以是js或者是jsx文件
│       └── Hello.css     // 组件的css样式，如果需要做样式的模块化则需要修改后缀，例如 Hello.model.css
├── App.css               // APP组件CSS样式
├── App.js                // APP组件(重要文件)
├── index.css             // 主页面样式
└── index.js              // 项目的入口文件（重要文件，类似于main函数）
~~~

### 结构二：使用组件时引入具体的组件所在目录，会自动查找该目录下的index组件和css（优点是引入组件省去写组件文件名称，缺点是多个组件目录下的index文件在写代码时可能会混淆）
~~~
src                       // 主要源码文件
├── components            // 组件目录，存放多个不同的组件
│   └── Hello             // 具体的组件目录，存放对应组件的js、jsx、css代码
│       ├── index.jsx     // 组件，可以是js或者是jsx文件
│       └── index.css     // 组件的css样式，如果需要做样式的模块化则需要修改后缀，例如 index.model.css
├── App.css               // APP组件CSS样式
├── App.js                // APP组件(重要文件)
├── index.css             // 主页面样式
└── index.js              // 项目的入口文件（重要文件，类似于main函数）
~~~

## Hello组件例子
### src/components/Hello.css Hello组件的css样式
~~~css
.title {
    background-color: aqua;
}
~~~

### src/components/Hello.jsx 
~~~jsx 
import React, { Component } from 'react'; 
import './Hello.css' // Hello组件的样式
// import hello from './Hello.model.css' // 样式的模块化，防止不同组件的样式冲突

export default class Hello extends Component {
    render() {
        return (
            <div className="title">hello word</div>
            // <div className={hello.title}>hello word</div> // 样式的模块化，防止不同组件的样式冲突
        )
    }
}
~~~

### src/App.js App组件引用Hello组件
~~~js
import React, { Component } from 'react';
import './App.css';
import Hello from './components/Hello/Hello'


export default class App extends Component {
  render() {
    return (
      <Hello />
    )
  }
}
~~~


