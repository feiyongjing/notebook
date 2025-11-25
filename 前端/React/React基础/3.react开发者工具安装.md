# react浏览器开发者工具react-devtools安装
1. 去 https://github.com/facebook/react-devtools/releases 查找对应浏览器需要安装的版本（这是v3版本过于老旧，会有问题）
2. 去官网 https://reactjs.bootcss.com/learn/react-developer-tools 查找最新版本安装

# vsCode安装插件 
### ES7+ React/Redux/React-Native snippets 插件作者是 dsznajders
1. 在空的组件文件中输入rcc回车快速生成react class component 类型的组件代码
2. 在空的组件文件中输入rfc回车快速生成react function component 类型的组件代码
3. 更多快捷代码片段参照 https://github.com/r5n-dev/vscode-react-javascript-snippets/blob/HEAD/docs/Snippets.md

# redux-devtools安装（用于查看redux存储所有组件数据）
- github下载插件手动安装 https://github.com/reduxjs/redux-devtools/releases
- 浏览器插件商店安装
- 安装完成之后还需要在项目中手动修改store 配置 createstore创建的最后一个参数，参考 https://blog.csdn.net/applebomb/article/details/54918659?utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-7.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~default-7.no_search_link

~~~js
// 引入createStore函数用于创建redux的核心对象store
// applyMiddleware函数是开启中间件，具体开启什么中间件就看参数了，这里是开启 thunk 异步中间件
// combineReducers函数是用于合并汇总所有的reducer
import { createStore, applyMiddleware, combineReducers, compose} from 'redux'

// 引入自定义的reducer，每一个reducer都专为一个组件进行服务
import count_reducer from './reducers/count_reducer'

// 引入thunk异步中间件
import {thunk} from 'redux-thunk'

// 合并汇总所有的reducer
// 注意后面通过 getState函数 获取的state是所有组件的数据，如果需要获取特点组件的数据还需要通过当前 reducer 的key来获取
const allReducer=combineReducers({
    count: count_reducer,
})


// 使reducer被store对象管理，整个应用只有一个store对象，它管理所有的reducer
// const store = createStore(allReducer, applyMiddleware(thunk))

// 判断插件是否安装，如果浏览器安装了就进行插件调试，没有就不进行插件调试而是正常加载
let store;
if(!(window.__REDUX_DEVTOOLS_EXTENSION__ || window.__REDUX_DEVTOOLS_EXTENSION__)){
    store = createStore( allReducer, applyMiddleware(thunk));
}else{
    store = createStore(
        allReducer,
        compose(applyMiddleware(thunk),window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()) //插件调试，未安装会报错
    );
}

// 暴露store对象
export default store;
~~~



