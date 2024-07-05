### 模板创建
~~~shell
vue3的vite详细创建项目参考: https://cn.vuejs.org/guide/quick-start.html#creating-a-vue-application

# Vite 需要 Node.js 版本 18+ 或 20+。然而，有些模板需要依赖更高的 Node 版本才能正常运行
# 使用的模板创建项目，npm 7+, 需要额外加 --:
# 常见模板：vanilla，vanilla-ts, vue, vue-ts，react，react-ts，react-swc，react-swc-ts，preact，preact-ts，lit，lit-ts，svelte，svelte-ts，solid，solid-ts，qwik，qwik-ts
npm create vite@latest [项目目录] -- --template vue

# 第一次需要手动下载依赖
npm i

# 运行命令启动项目服务，默认端口是5173
npm run dev
~~~
---

### 手动创建项目
~~~shell
# 手动创建项目
npm install -D vite

# 创建index.html文件
<p>Hello Vite!</p>

# 在index.html同一目录下运行vite，就可以在 http://localhost:5173 上访问 index.html
vite
~~~

# Vue3常见项目目录结构如下(与vue-cli的区别是index.html的位置不同)
/
├── public
│   ├── css
│   │   └── xxx.css
│   └── favicon.ico
├── src
│   ├── assets        放图片或css
│   │   └── test.png
│   ├── components    放一般组件
│   │   └── test.vue
│   ├── pages         放路由组件
│   │   └── Home.vue 
│   ├── router        放路由规则
│   │   └── index.js
│   ├── store         放vuex的配置和自定义的store模块
│   │   └── index.js
│   ├── js            放一些js代码
│   │   └── test.js
│   ├── App.vue       
│   └── main.js
├── .gitignore
├── index.html
├── babel.config.js
├── jsconfig.json
├── package-lock.json
├── package.json
├── vite.config.js    vite的配置
├── vue.config.js     vue的全局配置，可以配置后端服务的代理
└── README.md

### vue3的src/main.js启动入口
~~~js
// 引入的不再是Vue的构造函数了，而是一个名为createApp的工厂函数
import { createApp } from 'vue'
import App from './App.vue'

// 创建一个App实例对象（类似于Vue2的vm组件实例对象，但是比vm要更'轻'，即对象上的属性和方法要少一些）
// 并且App实例对象挂载到对应的Dom节点
createApp(App).mount('#app')
~~~




