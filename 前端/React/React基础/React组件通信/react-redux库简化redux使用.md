# react-redux库简化redux使用
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
├── components                              // 一般组件目录，存放多个不同的一般组件
│   └── Count                               // Count UI组件模块
│       └── Count.jsx                       // Count UI组件，可以是js或者是jsx文件
├── containers                              // 容器组件目录，存放多个不同的容器组件
│   └── Count                               // Count容器组件模块
│       └── index.jsx                       // Count容器组件，可以是js或者是jsx文件
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










