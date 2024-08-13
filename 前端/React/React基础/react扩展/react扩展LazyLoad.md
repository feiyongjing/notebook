# react扩展LazyLoad
- lazy函数懒加载加载路由组件
- Suspense组件设置当内部路由组件由于网络原因加载不出来时展示fallback属性设置的组件表示加载中

### 安装react-router-dom
- 注意react-router-dom版本5和6的使用完全不同
~~~shell
# 安装react-router-dom版本5
npm i react-router-dom@5
~~~

## react路由组件react-router-dom版本5使用
### 项目目录
~~~
src                             // 主要源码文件
├── components                  // 一般组件目录，存放多个不同的一般组件
│   ├── Loading                 // Loading组件模块
│   │   ├── Loading.jsx         // Loading组件，可以是js或者是jsx文件
│   │   └── Loading.model.css   // Loading组件模块化css样式
├── pages                       // 路由组件目录，存放多个不同的路由组件
│   ├── About                   // About路由组件模块
│   │   ├── About.jsx           // About路由组件，可以是js或者是jsx文件
│   │   └── About.model.css     // About路由组件模块化css样式
│   └── Home                   
│       ├── Home.jsx           
│       └── Home.model.css        
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
import React, { Component, lazy, Suspense } from 'react';
import { NavLink, Link, Route, Switch, Redirect } from 'react-router-dom';
import './App.css';

import Loading from './components/Loading/Loading'


// import About from './pages/About/About'
// import Home from './pages/Home/Home'
// 不直接引入加载路由组件，而是懒加载路由组件，防止在页面一次性加载太多的路由组件
const About = lazy(() => import("./pages/About/About.jsx"))
const Home = lazy(() => import("./pages/Home/Home.jsx"))

export default class App extends Component {

  render() {
    return (
      <div id="app">
        <h2>React Router Demo</h2>

        <NavLink activeClassName="aaa" className="bbb" to="/about">About</NavLink>
        <br />
        <NavLink activeClassName="aaa" className="bbb" to="/home/a/b">Home</NavLink>


        <Switch>
          {/* Suspense 设置当路由组件由于网络原因加载不出来时展示fallback属性设置的组件表示加载中 */}
          <Suspense fallback={<Loading />}>
            <Route path="/about" component={About} />
            <Route path="/home" component={Home} />
            <Redirect to="/home"></Redirect>
          </Suspense>

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

### src/pages/About/About.jsx About路由组件
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

### src/pages/Home/Home.jsx Home路由组件
~~~js
import React, { Component } from 'react'

export default class Home extends Component {

    render() {
        return (
            <div >
                 <h2>我是Home的内容</h2>
                 <input type="text"/>
            </div>
        )
    }
}

~~~

### src/components/Loading/Loading.jsx Loading一般组件
~~~js
import React, { Component } from 'react'

export default class Loading extends Component {
  render() {
    return (
      <div>加载中...</div>
    )
  }
}
~~~


