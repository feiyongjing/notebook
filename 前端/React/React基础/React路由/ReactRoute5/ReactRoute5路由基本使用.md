# react路由
- react-router-dom是专用于web网页SPA（单页面）路由的react官方插件库


### 安装react-router-dom
- 注意react-router-dom版本5和6的使用完全不同
~~~shell
# 安装react-router-dom版本5
npm i react-router-dom@5

# 安装react-router-dom版本6
npm i react-router-dom@6
~~~

## react路由组件react-router-dom版本5使用
### 项目目录
~~~
src                             // 主要源码文件
├── components                  // 一般组件目录，存放多个不同的一般组件
│   ├── Hello                   // Hello组件模块
│   │   ├── Hello.jsx           // Hello组件，可以是js或者是jsx文件
│   │   └── Hello.model.css     // Hello组件模块化css样式
├── pages                       // 路由组件目录，存放多个不同的路由组件
│   ├── About                   // About路由组件模块
│   │   ├── About.jsx           // About路由组件，可以是js或者是jsx文件
│   │   └── About.model.css     // About路由组件模块化css样式
│   ├── Home                   
│   │   ├── Home.jsx           
│   │   └── Home.model.css     
│   ├── News                   
│   │   ├── News.jsx           
│   │   └── News.model.css   
│   └── Message              
│       ├── Message.jsx     
│       └── Message.model.css     
├── App.css                     // APP组件CSS样式
├── App.js                      // APP组件(重要文件)
├── index.css                   // 主页面样式
└── index.js                    // 项目的入口文件（重要文件，类似于main函数）
~~~

### index.js 项目的入口文件 设置路由器
~~~js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import { BrowserRouter, HashRouter } from 'react-router-dom';  // BrowserRouter 和 HashRouter 都是路由器

const root = ReactDOM.createRoot(document.getElementById('root'));

// 必须确保所有的路由组件都被同一个路由器管理，否则路由链接不生效，所以只能使用一个路由器标签包裹App组件来管理所有的路由
// BrowserRouter 和 HashRouter 路由器的区别
// HashRouter路由器会在域名后的路径添加/#/ 会将/#/后面的路径当成路由路径去获取前端资源
// BrowserRouter路由器不会在域名后的路径添加/#/ 注意路由的路径页面刷新后直接拿着这个路径去获取后端资源而不是前端资源导致报错
// BrowserRouter路由器的解决方案是通过nginx配置路由转发来区分前端路由和后端路由进行代理转发
// 一般如果前后端API都是同一个域名是使用的 BrowserRouter 路由器
root.render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);

reportWebVitals();
~~~

### src/App.js APP组件 设置路由组件切换，也可以是在其他组件中进行路由组件切换
~~~js
import React, { Component } from 'react';
import { NavLink, Link, Route, Switch,Redirect } from 'react-router-dom';
import './App.css';

import About from './pages/About/About'
import Home from './pages/Home/Home'


export default class App extends Component {

  render() {
    return (
      <div id="app">
        <h2>React Router Demo</h2>

        {/* 原始html中我们使用a标签实现页面的跳转
        <a href="./about.html">About</a>
        <a href="./home.html">Home</a> */}

        {/* 在React中靠 Link组件标签或NavLink 设置路由路径实现切换路由组件，一般是在页面的导航菜单区编写路由链接
            Link组件标签无法设置点击时临时添加设置ClassName切换样式
            NavLink组件标签默认在点击时会添加一个叫active的class，但是也可以手动设置activeClassName来设置添加点击时的ClassName切换样式
            Link和 NavLink的push属性设置当前页面被浏览器记录（默认是push），而replace属性是设置当前页面及其子路由页面不被浏览器记录，
            即跳转到其他页面后无法回退到之前的页面，但是会继续向更早的页面跳转
         */}
        <NavLink activeClassName="aaa" className="bbb" to="/about">About</NavLink>
        <br />
        <NavLink activeClassName="aaa" className="bbb" to="/home/a/b">Home</NavLink>

        {/* 通过 Route组件标签 设置路由路径来注册实际的路由组件，一般路由组件都是在页面数据内容的展示区 

            通过 Switch组件标签 包裹路由注册，确保一个路径路由只有第一个路由组件会展示，后面其他的不展示
            如果不包裹Switch组件标签，当出现一个路由路由展示多个路由组件，这样会有效率问题
            如果只有一个路由组件可以省略 Switch组件标签 包裹

            通过 Redirect组件标签 设置当所有的路由都不匹配时默认选择路由进行展示路由组件

            默认Route中的path属性进行路由注册时是模糊匹配，即NavLink中的to属性值只要匹配Route中的path属性值前缀就可以展示该路由组件
            而通过设置Route中的exact属性表示开启严格匹配，必须要保证NavLink中的to属性值完整匹配Route中的path属性值才能展示该路由组件
            需要注意的是严格匹配不要随便开启，因为有时开启严格匹配会导致无法匹配多级路由
        */}
        <Switch>
          <Route path="/about" component={About} />
          <Route path="/home" component={Home} />
          <Redirect to="/home"></Redirect>
          {/* <Route path="/home" component={Test} /> */}
          {/* <Route exact path="/home" component={Home} /> */}
        </Switch>

      </div>

    )
  }
}
~~~

### src/App.css APP组件的css样式
~~~css
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  font-size: 20px;
  text-align: center;
}

.aaa {
  background-color: aqua;
}
~~~

### src/pages/About/About.jsx About一级路由组件
~~~js
import React, { Component } from 'react'
import "./About.css";

export default class About extends Component {

    render() {
        return (
            <div >
                <h2>我是About的内容</h2>
            </div>
        )
    }
}
~~~

### src/pages/Home/Home.jsx Home一级路由组件
~~~js
import React, { Component } from 'react'
import { NavLink, Route, Switch,Redirect } from 'react-router-dom';
import "./Home.css";
import News from '../News/News'
import Message from '../Message/Message'

export default class Home extends Component {

    render() {
        return (
            <div >
                <h2>我是Home的内容</h2>
                {/* 多级路由Link和注册的注意点就是路由的路径一定要写完整路径，
                    即链接和注册子路由时需要在父路由的path后面添加路径 */}
                <NavLink to="/home/news">News</NavLink>
                <br />
                <NavLink to="/home/message">Message</NavLink>

                <Switch>
                    <Route path="/home/news" component={News} />
                    <Route path="/home/message" component={Message} />
                    <Redirect to="/home/message"></Redirect>
                </Switch>
            </div>

        )
    }
}
~~~

### src/pages/News/News.jsx News二级路由组件
~~~js
import React, { Component } from 'react'

export default class News extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>news001</li>
                    <li>news002</li>
                    <li>news003</li>
                </ul>
            </div>
        )
    }
}
~~~

### src/pages/Message/Message.jsx Message二级路由组件
~~~js
import React, { Component } from 'react'

export default class Message extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li><a href="/message1">messages001</a>&nbsp;&nbsp;</li>
                    <li><a href="/message2">messages002</a>&nbsp;&nbsp;</li>
                    <li><a href="/message3">messages003</a>&nbsp;&nbsp;</li>
                </ul>
            </div>
        )
    }
}
~~~


