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

## react路由组件react-router-dom版本6使用
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


