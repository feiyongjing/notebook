# react-redux实现多组件通信
- 官网 https://cn.react-redux.js.org/

# react-redux使用注意事项
- 使用react-redux时都是将组件拆分为两个组件，一个是容器组件，另外一个是UI组件，并且容器组件包裹UI组件
- 容器组件负责和redux打交道，可以通过store操作redux的API来获取数据和更新数据
- UI组件不能直接通过store操作redux的API获取数据和修改数据，UI组件如果想要获取和更新redux中的数据需要通过调用容器组件中的方法来间接操作
- 容器组件通过props向UI组件传递获取和更新redux中数据的方法

### react-redux安装
~~~shell
# redux
npm install react-redux
yarn add react-redux
~~~

### 项目目录
~~~
src                                         // 主要源码文件
├── containers                              // 容器组件目录，存放多个不同的容器组件
│   │── Count                               // 容器组件模块
│   │   └── Count.jsx                       // 容器组件（内部包含对应的UI组件），可以是js或者是jsx文件
│   └── Person                               
│       └── Person.jsx                       
├── redux                                   // redux相关配置
│   │── reducers                            // 自定义不同组件的reducer，为这些组件服务
│   │   ├── count_reducer.js                
│   │   └── person_reducer.js               
│   │── actions                             // 自定义不同组件的action配置
│   │   ├── create_count_action.js                
│   │   └── person_count_action.js          
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
import { Provider } from 'react-redux' // 引入Provider组件

const root = ReactDOM.createRoot(document.getElementById('root'));

// Provider组件用于监听内部所有组件和后代组件，检测有哪些容器组件需要store，会将全局管理的store传递给需要的容器组件
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);

// 不用手动监听redux下store中的数据了，react-redux会在容器组件通过连接store对象连接redux时中自动进行监听数据修改和渲染
// store.subscribe(() => {
//   root.render(
//     <React.StrictMode>
//       <App />
//     </React.StrictMode>
//   );
// })

reportWebVitals();
~~~

###  src/App.js   APP组件
~~~js
import React, { Component } from 'react';
import CountContainer from './containers/Count/Count'
import PersonContainer from './containers/Person/Person'

export default class App extends Component {

  render() {

    return (
      <div>
        <CountContainer />
        <hr/>
        <PersonContainer />
      </div>
    )
  }
}
~~~

### src/containers/Count/Count.jsx   Count容器组件
~~~js
import React, { Component } from 'react'

// 引入react-redux的connect用于自定义的Count UI组件和redux的连接
import { connect } from 'react-redux'

// 引入自定义的action对象构建函数
import { createIncrementAction, createDecrementAction, createIncrementAsyncAction } from '../../redux/actions/create_count_action'

// CountUI组件
class CountUI extends Component {

    increment = () => {
        const value = this.selectNumber.value * 1
        // mapDispatchToProps编写方式一进行调用容器组件的函数
        // this.props.dispatch(createIncrementAction(value))

        // mapDispatchToProps编写方式二进行调用容器组件的函数
        this.props.jia(value)
    }

    decrement = () => {
        const value = this.selectNumber.value * 1
        // mapDispatchToProps编写方式一进行调用容器组件的函数
        // this.props.dispatch(createDecrementAction(value))

        // mapDispatchToProps编写方式二进行调用容器组件的函数
        this.props.jian(value)
    }

    incrementIfOdd = () => {
        // 从props中获取容器组件传递的redux存储数据（可以获取当前组件或者其他组件存储在redux中的数据）
        if (this.props.count % 2 === 0) {
            return
        }
        const value = this.selectNumber.value * 1
        // mapDispatchToProps编写方式一进行调用容器组件的函数
        // this.props.dispatch(createIncrementAction(value))

        // mapDispatchToProps编写方式二进行调用容器组件的函数
        this.props.jia(value)
    }

    incrementAsync = () => {
        const value = this.selectNumber.value * 1
        // mapDispatchToProps编写方式一进行调用容器组件的函数
        // this.props.dispatch(createIncrementAsyncAction(value, 1000))

        // mapDispatchToProps编写方式二进行调用容器组件的函数
        this.props.jiaAsync(value, 1000)
    }

    render() {

        // 从props中获取容器组件传递的redux存储数据（可以获取当前组件或者其他组件存储在redux中的数据）
        const {count, personObjs} = this.props

        return (
            <div>
                <h2>Count组件</h2>
                <h4>当前值是{count}</h4>
                <select ref={c => this.selectNumber = c}>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                </select>&nbsp;
                <button onClick={this.increment}>+</button>&nbsp;
                <button onClick={this.decrement}>-</button>&nbsp;
                <button onClick={this.incrementIfOdd}>当前值是奇数就加</button>&nbsp;
                <button onClick={this.incrementAsync}>异步加</button>&nbsp;

                <h2>Person组件数据展示在Count组件中</h2>
                <ul>
                    {
                        personObjs.map((personObj)=>{
                            return <li key={personObj.id}>用户名称：{personObj.name}，用户年龄：{personObj.age}</li>
                        })
                    }
                    
                </ul>
            </div>
        )
    }
}


// 参数是父组件（App组件）传递的store对象调用getState()获取redux中的数据，注意这里是获取的全部数据，即所有组件存放在redux中的数据
// 返回值作为向UI组件（CountUI）传递的 props 数据，需要rudex中哪些组件存储的数据就通过组件对应 reducer 的key（在核心store配置代码中）来获取取出哪些数据进行返回
function mapStateToProps(state) {
    return {
        count: state.count,
        personObjs: state.personObjs
    }
}

// 参数是父组件（App组件）传递的store中的dispatch函数，可以用于修改redux中的数据
// 返回值作为向UI组件（CountUI）传递的 props 函数
// mapDispatchToProps编写方式一：如下编写函数
// function mapDispatchToProps(dispatch) {
//     return {
//         dispatch: (action)=> dispatch(action)
//     }
// }

// mapDispatchToProps编写方式二：如下编写对象
// 本质是使用key，代理了action对象的创建和dispatch函数调用修改redux中的数据
// 所以使用key时传递的参数与action创建时传递的参数一致
const mapDispatchToProps = {
    jia: createIncrementAction,
    jian: createDecrementAction,
    jiaAsync: createIncrementAsyncAction
}


// 创建一个 Count 容器组件，包裹CountUI 组件，并且向它传递props数据和函数
export default connect(mapStateToProps, mapDispatchToProps)(CountUI)
~~~

### src/containers/Person/Person.jsx   Person容器组件
~~~js
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { createAddPersonAction } from '../../redux/actions/create_person_action'

class PersonUI extends Component {


    addPerson=()=>{
        const id = this.props.personObjs.length+1
        const name=this.nameNode.value
        const age=this.ageNode.value

        this.props.add({id, name, age})
    }


    render() {
        const {count, personObjs} = this.props

        return (
            <div>
                <h2>Person组件</h2>
                <input ref={c => this.nameNode = c} placeholder='请输入名称' />
                <input ref={c => this.ageNode = c} placeholder='请输入年龄' />
                <button onClick={this.addPerson}>添加</button>
                <ul>
                    {
                        personObjs.map((personObj)=>{
                            return <li key={personObj.id}>用户名称：{personObj.name}，用户年龄：{personObj.age}</li>
                        })
                    }
                    
                </ul>

                <h2>Count组件数据展示在Person组件中</h2>
                <h4>当前值是{count}</h4>
            </div>
        )
    }
}


function mapStateToProps(state) {
    return {
        count: state.count,
        personObjs: state.personObjs
    }
}

const mapDispatchToProps = {
    add: createAddPersonAction
}


// 创建一个 Count 容器组件，包裹PersonUI 组件，并且向它传递props数据和函数
export default connect(mapStateToProps, mapDispatchToProps)(PersonUI)
~~~

### src/redux/store.js   redux的核心store配置
~~~js
// 引入createStore函数用于创建redux的核心对象store
// applyMiddleware函数是开启中间件，具体开启什么中间件就看参数了，这里是开启 thunk 异步中间件
// combineReducers函数是用于合并汇总所有的reducer
import { createStore, applyMiddleware, combineReducers } from 'redux'

// 引入自定义的reducer，每一个reducer都专为一个组件进行服务
import count_reducer from './reducers/count_reducer'
import person_reducer from './reducers/person_reducer'

// 引入thunk异步中间件
import {thunk} from 'redux-thunk'

// 合并汇总所有的reducer
// 注意后面通过 getState函数 获取的state是所有组件的数据，如果需要获取特点组件的数据还需要通过当前 reducer 的key来获取
const allReducer=combineReducers({
    count: count_reducer,
    personObjs: person_reducer
})

// 使reducer被store对象管理，整个应用只有一个store对象，它管理所有的reducer
const store = createStore(allReducer, applyMiddleware(thunk))

// 暴露store对象
export default store;
~~~

### src/redux/reducers/count_reducer.js   自定义Count组件reducer，专为Count组件服务
~~~js
import {INCREMENT, DECREMENT} from '../actions/create_count_action' // 引入自定义的action操作常量

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

### src/redux/reducers/person_reducer.js   自定义Person组件reducer，专为Person组件服务
~~~js
import {ADD_PERSON} from '../actions/create_person_action' // 引入自定义的action操作常量

/**
 * reducer 进行实际的数据修改
 * @param {*} oldState 表示之前的state数据，如果是第一次或者不传递就是undefined，会使用默认参数
 * @param {*} action 动作对象，内部有type和data表示需要进行的操作类型和传递的数据
 * @returns 新的state数据
 */
const initState = [{id: "001", name:"张三", age:19}] // 初始化的数据
export default function person_reducer(oldState=initState, action) {
    const { type, data } = action
    switch (type) {
        case ADD_PERSON:
            // oldState.unshift(data) 此次不能这样写，原因是必须返回一个新的对象（新地址），才会真正的修改state的数据
            // 主要原因是 reducer 函数必须保证是一个纯函数，纯函数不允许修改传入的参数
            return [data, ...oldState]
        default:
            return oldState
    }

}
~~~

### src/redux/actions/create_count_action.js   Count组件有关的action对象创建函数
~~~js
// 定义不同的action操作常量
export const INCREMENT="increment"
export const DECREMENT="decrement"

// 设置不同的action对象构建函数
export const createIncrementAction= data => ({ type: INCREMENT, data})
export const createDecrementAction= data => ({ type: DECREMENT, data})

// 异步Action：Action的值是一个函数，并且在异步Action中一般会调用dispatch函数修改数据，参数是调用同步的Action对象
export const createIncrementAsyncAction = (data, time) => {
    return (dispatch)=>{
        setTimeout(() => {
            dispatch(createIncrementAction(data))
        }, time)
    }
}
~~~

### src/redux/actions/person_count_action.js   Person组件有关的action对象创建函数
~~~js
// 定义不同的action操作常量
export const ADD_PERSON="addPerson"

// 设置不同的action对象构建函数
export const createAddPersonAction= personObj => ({ type: ADD_PERSON, data: personObj})
~~~


