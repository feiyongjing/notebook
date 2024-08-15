# react路由参数使用

### 项目目录
- 以下几种路由参数使用都是这个项目结构
~~~
src                             // 主要源码文件
├── components                  // 一般组件目录，存放多个不同的一般组件
│   ├── Message                 // Message组件模块
│   │   ├── Message.jsx         // Message组件，可以是js或者是jsx文件
│   │   └── Message.model.css   // Message组件模块化css样式
├── pages                       // 路由组件目录，存放多个不同的路由组件
│   └── Detail                  // About路由组件模块
│       ├── Detail.jsx          // About路由组件，可以是js或者是jsx文件
│       └── Detail.model.css    // About路由组件模块化css样式   
├── App.css                     // APP组件CSS样式
├── App.js                      // APP组件(重要文件)
├── index.css                   // 主页面样式
└── index.js                    // 项目的入口文件（重要文件，类似于main函数）
~~~

### src/App.js APP组件 
~~~js
import React, { Component } from 'react';
import './App.css';

import Message from './component/Message/Message'

export default class App extends Component {

  render() {
    return (
      <div id="app">
        <h2>React Router Demo</h2>
        <Message/>
      </div>
    )
  }
}
~~~

### src/components/Message/Message.jsx Message组件向路由组件传递参数，并且将Message这个一般组件加工成拥有路由组件API的特殊组件 
~~~js
import React, { Component } from 'react'
import { NavLink, Route, withRouter } from 'react-router-dom';
import Detail from '../../pages/Detail/Detail'

class Message extends Component {

    state = {
        messageArr: [
            { id: "001", title: "消息1", content: "消息1内容" },
            { id: "002", title: "消息2", content: "消息2内容" },
            { id: "003", title: "消息3", content: "消息3内容" },
        ]
    }

    pushShow = (id, title, content) => {
        this.props.history.push(`/detail`, { id, title, content })
    }

    replaceShow = (id, title, content) => {
        this.props.history.replace(`/detail`, { id, title, content })
    }

    goBack = () => {
        this.props.history.goBack()
    }

    goForward = () => {
        this.props.history.goForward()
    }

    render() {
        const { messageArr } = this.state
        return (
            <div>
                <ul>
                    {
                        messageArr.map((message) => {
                            const { id, title, content } = message
                            return (
                                <li key={message.id}>
                                    {/* 传递具体的path params参数（路径参数）给路由组件 */}
                                    {/* <NavLink to={`/detail/${id}/${title}/${content}`}>{title}</NavLink> */}

                                    {/* 传递具体的query params参数给路由组件 */}
                                    {/* <NavLink to={`/detail/?id=${id}&title=${title}&content=${content}`}>{title}</NavLink> */}

                                    {/* 传递具体的body params参数给路由组件 */}
                                    <NavLink to={{ pathname: '/detail', state: { id, title, content } }}>{title}</NavLink>

                                    {/* 编程式路由导航，即手动触发事件进行路由导航，但是注意当前组件必须是路由组件，否则this.props中没有history 
                                        如果当前组件不是路由组件，请使用withRouter将其加工成拥有路由组件API的特殊组件  */}
                                    <button onClick={() => this.pushShow(id, title, content)} >push跳转</button>
                                    <button onClick={() => this.replaceShow(id, title, content)} > replace跳转</button> 
                                </li>
                            )
                        })
                    }
                </ul >

                {/* 设置path params参数列表（路径参数列表），通过路径参数向路由组件传递参数
                    路由组件可以在 props中match下的params获取路径参数
                 */}
                {/* <Route path="/detail/:id/:title/:content" component={Detail} /> */}

                {/* query params参数无需声明接收形参列表，原因是在传递参数给路由组件时已经声明了形参
                    路由组件可以在 props中location获取query参数
                 */}
                {/* <Route path="/detail" component={Detail} /> */}

                {/* body params参数也无需声明接收形参列表，原因是在传递参数给路由组件时已经声明了形参 
                    路由组件可以在 props中location下的state获取body参数
                */}
                <Route path="/detail" component={Detail} />


                {/* 路由的回退与前进，注意当前组件必须是路由组件，否则this.props中没有history 
                    如果当前组件不是路由组件，请使用withRouter将其加工成拥有路由组件API的特殊组件   */}
                <button onClick={this.goBack} >后退</button>
                <button onClick={this.goForward} >前进</button>
            </div >
        )
    }
}

// 通过withRouter将一般组件可以加工成拥有路由组件API的特殊组件 
export default withRouter(Message)
~~~

### src/pages/Detail/Detail.jsx Detail路由组件
~~~js
import React, { Component } from 'react'
import qs from 'qs'

export default class Detail extends Component {

    render() {

        console.log(this.props)

        // 路由组件可以在 props中match下的params获取路径参数
        // const { id, title, content} = this.props.match.params

        // 路由组件可以在 props中location获取query参数
        // const { search } = this.props.location
        // const { id, title, content } = qs.parse(search.slice(1))

        // 路由组件可以在 props中location下的state获取body参数
        const { id, title, content } = this.props.location.state 

        return (
            <ul>
                <li >ID号：{id}</li>
                <li >消息标题：{title}</li>
                <li >消息详情：{content}  </li>
            </ul>
        )
    }
}

~~~
---




