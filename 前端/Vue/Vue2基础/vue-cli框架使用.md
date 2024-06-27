# vue-cli创建和启动项目
~~~shell
# 安装vue-cli
npm install -g @vue/cli

# 将npm设置为淘宝镜像
npm config set registry https://registry.npm.taobao.org

# windows下git-bash创建项目，执行会让你选择使用vue2还是vue3创建项目
winpty vue.cmd create 项目名
# windows的cmd 下创建项目，执行会让你选择使用vue2还是vue3创建项目
vue create 项目名

# 进入创建的项目
cd 项目名
# 
# public目录存放index.html和css、favicon.ico页签图标
# src目录下有main.js启动入口和App.vue组件（管理所有组件的祖先组件）
# src/components目录下放自定义的vue组件代码
# src/assets目录下放img图片等静态资源
# vue-cli会将webPack的配置默认隐藏起来，如果需要查看请在项目下执行 vue inspect > output.json 将配置打印到output.json文件中c查看
# 如果需要修改上述的css、App.vue、src/assets的目录或名称（注意项目下的public目录、public/index.html、public/favicon.ico、src目录、src/main.js这5个文件或者目录名称不能修改），请在项目根目录下创建vue.config.js进行修改，可以修改的属性参考：https://cli.vuejs.org/zh/config/

# 常见项目目录结构如下
/
├── public
│   ├── css
│   │   └── xxx.css
│   ├── index.html
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
├── babel.config.js
├── jsconfig.json
├── package-lock.json
├── package.json
├── vue.config.js     vue的全局配置，可以配置后端服务的代理
└── README.md

# 安装项目依赖，启动项目，默认是8080端口
npm install
npm run serve

# 如果需要自定义脚手架请在package.json文件同级目录下创建vue.config.js文件
# vue.config.js文件配置参考https://cli.vuejs.org/zh/config/
# 在修改vue.config.js文件配置后注意重新npm run serve启动项目

# npm run serve启动之后就会去执行项目下src/main.js
~~~

### 项目中package.js简介
~~~
{
  "name": "项目名称",
  "version": "0.1.0", // 项目版本
  "private": true,
  "scripts": {        // npm run [指令] 运行时会运行这里面的指令
    "serve": "vue-cli-service serve",    // npm run server 启动项目默认端口是8080
    "build": "vue-cli-service build",    // npm run build 打包出dist文件
    "lint": "vue-cli-service lint"       // npm run lint 进行代码语法相关检查
  },
  ...
}
~~~

### 默认的index.html
~~~html
<!DOCTYPE html>
<html lang="">

<head>
  <meta charset="utf-8">
  <!-- 针对IE浏览器的特殊配置，含义是让IE浏览器以最高的渲染级别渲染页面 -->
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <!-- 开启移动端的理想视口 -->
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <!-- 配置页签图标，<%= BASE_URL %>代表public目录  -->
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  <!-- 配置网页标题 <%= htmlWebpackPlugin.options.title %>代表 package.json中的name -->
  <title>
    <%= htmlWebpackPlugin.options.title %>
  </title>
</head>

<body>
  <!-- noscript标签是当浏览器不支持js时会将标签中的内容就会被渲染到页面上，基本没有这样的浏览器（这样的浏览器都成了时代的眼泪） -->
  <noscript>
    <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled.
        Please enable it to continue.</strong>
  </noscript>
  <!-- 容器 -->
  <div id="app"></div>
</body>

</html>
~~~

### main.js代码简介
~~~js
// 该文件是整个项目的入口文件
// 引入Vue，但是引入的残缺版的Vue，该vue是不包含模板解析器的，所以需要使用reande函数自行渲染模板
// 在node_modules下的vue包下的package.json中的属性"module": "dist/vue.runtime.esm.js"指向的残缺vue，
// 完整的Vue代码在dist/vue.js
// import Vue from 'vue/dist/vue'
// 不使用完整版的vue的原因是完整版的vue带有的模板解析器代码过多,而在生产环境下使用的代码是webpack打包好的js代码，无需模板解析器解析vue文件到js代码
import Vue from 'vue'
// 引入App组件，它是所有组件的父组件
import App from './App'

// 关闭Vue的生产提示
Vue.config.productionTip = false

// 创建Vm，vm管理App容器，并将App组件放入App容器
const vm=new Vue({
  // reande函数是用于渲染模板
  render: h => h(App),
}).$mount('#app')
~~~

### App.vue引入其他组件注册，例子中是引入自定义的School组件注册
~~~vue
<template>
  <div>
    <div class="row">
      <!-- 组件标签可以写单标签<School/>  -->
      <School></School> 
    </div>
  </div>
</template>

<script>
import School from "./components/School.vue";
export default {
  name: "App",
  components: {
    School,  // 由于School: School的key和value都一样，所以可以简写为School
  },
};
</script>

<style scoped>
</style>
~~~

### 子定义的School组件
~~~Vue
<template>
    <div class ="school">
           <h3>学校名称：{{schoolName}}</h3>
           <h3>学校地址：{{address}}</h3>
           <button @click="showName">点我弹出学校名称</button>
    </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      schoolName: "育才小学",
      address: "xxx省xxx市xxx区xxx街道23号",
    };
  },
  methods: {
    showName() {
      alert(this.schoolName);
    },
  },

//   components: {}, 用于继续引入子组件注册
};
</script>

<style>
.school {
  background-color: aquamarine;
}
</style>
~~~

### vue.config.js使用例子，默认情况下没有该配置文件，注意不要写一些重要配置设置空的对象或者注释掉，这样会导致出现问题
~~~js
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  lintOnSave: false, //关闭语法检测，一些创建后没有使用的函数或者变量会导致语法检查不过
  transpileDependencies: true,
  
  pages: {
    index: {
      // 程序的入口路径
      entry: 'src/main.js',
      // 模板来源路径
      template: 'public/index.html',
    },
  }
})
~~~




