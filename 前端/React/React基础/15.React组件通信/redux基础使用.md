# redux基础使用
- 官网 https://cn.redux.js.org/

### 安装redux
~~~shell
# redux
npm install redux
yarn add redux
~~~

### 项目目录
~~~
src                                         // 主要源码文件
├── components                              // 一般组件目录，存放多个不同的一般组件
│   └── Count                               // Count组件模块
│       ├── Count.jsx                       // Count组件，可以是js或者是jsx文件
│       └── Count.model.css                 // Count组件模块化css样式  
├── redux                                   // redux相关配置
│   │── count                               // 组件相关的redux配置
│   │   ├── count_reducer.js                // 自定义Count组件reducer，专为Count组件服务
│   │   └── create_count_action.js          // 与组件有关的action对象创建函数
│   └── store.js                            // redux的核心store配置
├── App.css                                 // APP组件CSS样式
├── App.js                                  // APP组件(重要文件)
├── index.css                               // 主页面样式
└── index.js                                // 项目的入口文件（重要文件，类似于main函数）
~~~


### src/index.js   项目的入口文件
~~~js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import store from './redux/store' // 自己创建的redux核心对象store

const root = ReactDOM.createRoot(document.getElementById('root'));


root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// 监听redux下store中的数据方式二
// 由于每个组件都设置componentDidMount生命周期手动订阅redux下store中的数据修改并且设置回调重新进行渲染页面比较麻烦
// 所以直接在App组件上进行监听redux下store中的数据并且重新渲染App组件，这样会同时渲染App下的所有组件
// 至于渲染App和下面的所有组件的效率问题则是交给React的Diff算法来决定局部刷新
store.subscribe(() => {
  root.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
})

reportWebVitals();
~~~

### src/App.js   APP组件
~~~js
import React, { Component } from 'react';
import Count from './components/Count/Count'

export default class App extends Component {

  render() {
    
    return (
      <Count />
    )
  }
}
~~~

### src/components/Count/Count.jsx   Count组件
~~~js
import React, { Component } from 'react'
// 自己创建的redux核心对象store
import store from '../../redux/store' 

// 引入自定义的action对象构建函数
import {createIncrementAction, createDecrementAction} from '../../redux/count/create_count_action'

export default class Count extends Component {

    // 监听redux下store中的数据方式一
    // 通过store的dispatch只会修改数据，不同像setState修改数据之后自动调用render函数重新渲染页面
    // 所以需要在组件挂载完毕生命周期钩子中订阅redux下store中的数据数据修改，设置修改的回调进行渲染页面
    // componentDidMount(){
    //     store.subscribe(()=>{
    //         // 由于render手动调用不生效
    //         // 所以使用setState但是不传入更新数据，但是会间接的调用render函数重新渲染页面
    //         this.setState({})
    //     })
    // }


    increment = () => {
        const value = this.selectNumber.value * 1
        // store的dispatch函数修改数据，参数是action对象（包含操作类型和传递的数据）
        store.dispatch(createIncrementAction(value))
    }

    decrement = () => {
        const value = this.selectNumber.value * 1
        store.dispatch(createDecrementAction(value))
    }

    incrementIfOdd = () => {
        // store的getState函数获取redux全部数据，然后根据组件对应 reducer 的key（在核心store配置代码中）来获取该组件存储的数据
        if (store.getState().count % 2 === 0) {
            return
        }
        const value = this.selectNumber.value * 1
        store.dispatch(createIncrementAction(value))
    }

    incrementAsync = () => {
        const value = this.selectNumber.value * 1
        setTimeout(() => {
            store.dispatch(createIncrementAction(value))
        }, 1000)

    }

    render() {
        return (
            <div>
                <h1>当前值是{store.getState().count}</h1>
                <select ref={c => this.selectNumber = c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前值是奇数就加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;
            </div>
        )
    }
}
~~~

### src/redux/store.js   redux的核心store配置
~~~js
// 引入createStore函数用于创建redux的核心对象store
// combineReducers函数是用于合并汇总所有的reducer
import { createStore, combineReducers } from 'redux'

// 引入自定义的reducer，每一个reducer都专为一个组件进行服务（这里是Count组件）
import count_reducer from './count/count_reducer' 

// 合并汇总所有的reducer
// 注意后面通过 getState函数 获取的state是所有组件的数据，如果需要获取特点组件的数据还需要通过当前 reducer 的key来获取
const allReducer=combineReducers({
    count: count_reducer
})

// 使reducer被store对象管理，整个应用只有一个store对象，它管理所有的reducer
const store =createStore(allReducer)

// 暴露store对象
export default store;
~~~

### src/redux/count/count_reducer.js   自定义Count组件reducer，专为Count组件服务
~~~js
import {INCREMENT, DECREMENT} from './create_count_action' // 引入自定义的action操作常量

/**
 * reducer 进行实际的数据修改
 * @param {*} oldState 表示之前的state数据，如果是第一次或者不传递就是undefined，会使用默认参数
 * @param {*} action 动作对象，内部有type和data表示需要进行的操作类型和传递的数据
 * @returns 新的state数据
 */
const initState = 0 // 初始化的数据
export default function count_reducer(oldState=initState, action) {
    const { type, data } = action
    switch (type) {
        case INCREMENT:
            return oldState + data
        case DECREMENT:
            return oldState - data
        default:
            return oldState
    }

}
~~~

### src/redux/count/create_count_action.js   与组件有关的action对象创建函数
~~~js
// 定义不同的action操作常量
export const INCREMENT="increment"
export const DECREMENT="decrement"

// 设置不同的action对象构建函数
export const createIncrementAction= data => ({ type: INCREMENT, data})
export const createDecrementAction= data => ({ type: DECREMENT, data})
~~~






