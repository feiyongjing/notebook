# ReactRoute6路由基本使用

### 安装react-router-dom
- 注意react-router-dom版本5和6的使用完全不同
~~~shell
# 安装react-router-dom版本5
npm i react-router-dom@5

# 安装react-router-dom版本6
npm i react-router-dom@6
~~~

## react路由组件react-router-dom版本6使用
- react-router-dom版本6推荐使用函数式组件写法，如下是函数式的写法

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
import React from 'react';
import { NavLink, Link, Route, Routes, Navigate, useRoutes } from 'react-router-dom';
import './App.css';

import About from './pages/About/About'
import Home from './pages/Home/Home'
import News from './pages/News/News'
import Message from './pages/Message/Message'

export default function App() {


  const computedClassname = ({ isActive }) => {
    // console.log("是否匹配当前路径", isActive)
    return isActive ? "aaa" : "bbb"
  }

  return (
    <div id="app">
      <h2>React Router Demo</h2>

      {/* 原始html中我们使用a标签实现页面的跳转
    <a href="./about.html">About</a>
    <a href="./home.html">Home</a> */}

      {/* 在React中靠 Link组件标签或NavLink 设置路由路径实现切换路由组件，一般是在页面的导航菜单区编写路由链接
        Link组件标签无法设置点击时临时添加设置ClassName切换样式
        NavLink组件标签可以设置className属性是个函数，函数的默认参数是个对象，对象有个isActive属性表示是否匹配当前路径，
        判断isActive属性进行设置class属性实现点击时的样式切换
        NavLink组件标签可以写end属性，表示匹配当子路由时isActive就变为false（默认不写end在匹配子路由isActive还是true），通过这个来失去样式


        Link和 NavLink的push属性设置当前页面被浏览器记录（默认是push），而replace属性是设置当前页面及其子路由页面不被浏览器记录，
        即跳转到其他页面后无法回退到之前的页面，但是会继续向更早的页面跳转
     */}
      <NavLink className={computedClassname} to="/about">About</NavLink>
      <br />
      <NavLink className={computedClassname} end to="/home">Home</NavLink>

      {/* 方式一：手动编写标签完成路由表，即路由路径对应路由组件 */}
      {/* 通过 Routes组件标签 包裹路由注册，确保一个路径路由只有第一个路由组件会展示，后面其他的不展示
        Route组件标签必须被Routes组件标签包裹
    
        通过 Route组件标签 设置路由路径来注册实际的路由组件，一般路由组件都是在页面数据内容的展示区 
        可以通过在 Route组件标签 设置caseSensitive属性设置区分大小写，默认不加是不区分大小写

        通过 Navigate组件标签 渲染指定路由路径的路由组件
      */}
      {/* <Routes>
        <Route path="/about" element={<About />} />
        <Route path="/home" element={<Home />} />
        <Route path="/" element={<Navigate to="/home" />} />
        </Routes> */}

      {/* 方式二：通过useRoutes函数传入路由表对象生成路由表，并且可以将子路由一并编写
          注意一般在项目中是将路由表对象放在 src/routes/ 目录之下的文件中，然后引用不同的路由表对象
      */}
      {
        useRoutes(
          [
            {
              path: "/about",
              element: <About />
            },
            {
              path: "/home",
              element: <Home />,
              children: [
                {
                    // 子路由直接写路径名，会默认带上父路由路径前缀的，但是不用在最前面写/，如果想要表示父路由下的根目录就用 ./ 
                    // 在路由组件（Home组件）直接通过 Outlet标签 设置子路由组件（News或者Message组件）出现的位置（类似于插槽功能）
                    path: "news",
                    element: <News />
                },
                {
                    path: "message",
                    element: <Message />
                },
                {
                    path: "./",
                    element: <Navigate to="/home/message" />
                }
            ]
            },
            {
              path: "/",
              element: <Navigate to="/home" />
            }
          ]
        )
      }

    </div>
  )
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

.bbb {
  background-color: orange;
}
~~~

### src/pages/About/About.jsx About一级路由组件
~~~js
import React, { useState } from 'react'
import { Navigate } from 'react-router-dom';

export default function About() {

    const [sum, setSum] = useState(1)

    return (
        <div >
            <h2>我是About的内容，sum的值是{sum}，如果sum的值变为2就进行路由跳转到home</h2>
            {sum === 2 ?
                <Navigate to="/home" />
                :
                <button onClick={() => { setSum(2) }}>点我sum变为2</button>
            }
        </div>
    )
}

~~~

### src/pages/Home/Home.jsx Home一级路由组件
~~~js
import React from 'react'
import { NavLink, Outlet } from 'react-router-dom';

export default function Home() {
    return (
        <div >
            <h2>我是Home的内容</h2>
            {/* 多级路由的路径不用写完整路径，而是直接写当前路由的路径（默认会自动继承父路由前缀的），
                注意多级路由路径不要在开头写/，而是要么直接写 路径 或者 ./路径
                如下多级路由路径直接写 news 或者 ./message */}
            <NavLink to="news">News</NavLink>
            <br />
            <NavLink to="./message">Message</NavLink>

            {/* Outlet标签设置路由标签注册出现的位置（和插槽的作用类似） */}
            <Outlet/>
        </div>
    )
}
~~~

### src/pages/News/News.jsx News二级路由组件
~~~js
import React from 'react'

export default function News() {
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
~~~

### src/pages/Message/Message.jsx Message二级路由组件
~~~js
import React from 'react'

export default function Message() {
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
~~~

