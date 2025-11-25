# ReactRoute6路由参数使用

### 项目目录
- 以下几种路由参数使用都是这个项目结构
~~~
src                             // 主要源码文件
├── pages                       // 路由组件目录，存放多个不同的路由组件
│   └── Message                  // About路由组件模块
│       ├── Message.jsx          // About路由组件，可以是js或者是jsx文件
│       └── Message.model.css    // About路由组件模块化css样式   
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

### src/App.js APP组件   APP组件向Message路由组件传递参数
~~~js
import React, { useState } from 'react';
import { NavLink, useNavigate, useRoutes } from 'react-router-dom';
import './App.css';

import Message from './pages/Message/Message'

export default function App() {

  // 通过 useNavigate 函数返回的 navigate对象来进行编程式的路由导航
  // 使用 useNavigate 就无需再像route5那样对一般组件使用withRouter加工组件成为路由组件
  const navigate = useNavigate()

  const [messageArr, setMessageArr] = useState(
    [
      { id: "001", title: "消息1", content: "消息1内容" },
      { id: "002", title: "消息2", content: "消息2内容" },
      { id: "003", title: "消息3", content: "消息3内容" },
    ]
  )


  const computedClassname = ({ isActive }) => {
    // console.log("是否匹配当前路径", isActive)
    return isActive ? "aaa" : "bbb"
  }

  // 带历史记录的跳转，如果需要传递params或者query参数就手动拼接到路由路径中
  const pushShow = (id, title, content) => {
    
    navigate(`/message`, {
      replace: false,
      state: {
        id, title, content
      }
    })

  }

  // 不带历史记录的跳转
  const replaceShow = (id, title, content) => {
    console.log(id,title,content)
    navigate(`/message`, {
      replace: true,
      state: {
        id, title, content
      }
    })
  }

  // 通过正负数设置后退多少步跳转
  const goBack = () => {
    navigate(-1)
  }

  // 通过正负数设置前进多少步跳转
  const goForward = () => {
    navigate(1)
  }

  return (
    <div id="app">
      <h2>React Router Demo</h2>

      <ul>
        {
          messageArr.map((message) => {
            const { id, title, content } = message
            return (
              <li key={message.id}>

                {/* 传递具体的path params参数（路径参数）给路由组件 */}
                {/* <NavLink className={computedClassname} to={`/message/${id}/${title}/${content}`}>{title}正常跳转</NavLink> */}

                {/* 传递具体的query params参数给路由组件 */}
                {/* <NavLink className={computedClassname} to={`/message/?id=${id}&title=${title}&content=${content}`}>{title}正常跳转</NavLink> */}

                {/* 传递具体的body params参数给路由组件 */}
                <NavLink className={computedClassname} to={'/message'} state={{ id, title, content }}>{title}正常跳转</NavLink>


                {/* 编程式路由导航，即手动触发事件进行路由导航  */}
                <button onClick={()=> pushShow(id, title, content)} >push模式路由跳转</button>
                <button onClick={()=> replaceShow(id, title, content)} > replace模式路由跳转</button>
              </li>
            )
          })
        }
      </ul >

      {
        useRoutes(
          [
            // 设置path params参数列表（路径参数列表），通过路径参数向路由组件传递参数
            // 路由组件可以通过 useParams函数 直接获取传入路径组件的路径参数
            // {
            //   path: "/message/:id/:title/:content",
            //   element: <Message />
            // }

            // query params参数无需声明接收形参列表，原因是在传递参数给路由组件时已经声明了形参
            // 路由组件可以通过 useSearchParams函数 获取传入路径组件的query参数
            // {
            //   path: "/message",
            //   element: <Message />
            // }

            // body params参数也无需声明接收形参列表，原因是在传递参数给路由组件时已经声明了形参 
            // 路由组件可以在 useLocation()函数返回的location对象下的state属性获取body参数 
            {
              path: "/message",
              element: <Message />
            }

          ]
        )
      }

      {/* 路由的回退与前进，注意当前组件必须是路由组件  */}
      <button onClick={goBack} >后退</button>
      <button onClick={goForward} >前进</button>

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

### src/pages/Message/Message.jsx 
~~~js
import React from 'react'
import { useParams, useSearchParams,useLocation,useNavigationType} from 'react-router-dom';

export default function Message() {

    // 路由组件可以通过 useParams函数 直接获取传入路径组件的路径参数
    // const { id, title, content } = useParams()

    // 路由组件可以通过 useSearchParams函数 获取传入路径组件的query参数
    // 注意调用 useSearchParams函数 返回的是一个两元素的数组，第一个元素是query参数，第二个参数是修改query的函数
    // query参数中的元素必须通过get方法手动获取
    // const [query, setQuery] = useSearchParams()
    // const id =query.get("id")
    // const title =query.get("title")
    // const content =query.get("content")

    // 路由组件可以在 useLocation()函数返回的location对象下的state属性获取body参数
    const { id, title, content } = useLocation().state

    // POP是浏览器直接打开，push和replace表示记录和不记录的路由跳转
    console.log("当前路由组件是如何打开的：",useNavigationType())

    return (
        <div>
            <ul>
                <li >ID号：{id}</li>
                <li >消息标题：{title}</li>
                <li >消息详情：{content}  </li>
            </ul>
        </div>
    )
}
~~~






