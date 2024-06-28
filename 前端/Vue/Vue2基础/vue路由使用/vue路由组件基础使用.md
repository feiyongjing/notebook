# 为什么使用vue-router插件
Vue 适合构建单页面应用。对于大多数此类应用，都推荐使用官方支持的Vue Router
在单页面应用(Single-page application）中，客户端的 JavaScript 可以拦截页面的跳转请求，动态获取新的数据，然后在无需重新加载的情况下更新当前页面。这样通常可以带来更顺滑的用户体验，因为这类场景下用户通常会在很长的一段时间中做出多次交互。这类的单页面应用中，路由的更新是在客户端执行的。


### 安装vue-router插件
~~~shell
# 对于vue2，我们推荐使用vue-router 3.x版本（无法使用4.x版本）。若vue3使用vue-router 4.x版本
# 默认是安装的4.x版本，所以需要指定一下版本
npn i vue-router@3.5.1
~~~

## 使用vue-router插件设置路由

### 项目目录如下
- src/router目录下放路由组件，src/components目录下放一般组件（当然当前案例没有使用一般组件）
~~~shell
/
└── src
    ├── components
    │   └── xxx.vue
    ├── pages
    │   ├── About.vue 
    │   └── Home.vue 
    ├── router
    │   └── index.js
    ├── App.vue
    └── main.js
~~~

### src/main.js 引入vue-router插件应用并且设置自定义的路由规则（路由器）
~~~js 
import Vue from 'vue'
import App from './App'
import VueRouter from 'vue-router'  // 引入vue-router插件
import router from './router'       // 引入路由规则（路由器），路由规则一般写在router目录下，如果是在index.js文件就只需要提供目录即可，默认会在目录下查找index.js文件

// 应用vue-router插件
Vue.use(VueRouter)

// 关闭Vue的生产提示
Vue.config.productionTip = false

const vm=new Vue({
  // router:router 使用自己编写的路由规则，简写为router
  router,
  render: h => h(App),
}).$mount('#app')
~~~

### src/router/index.js 自定义的路由规则（路由器）
~~~js
import VueRouter from 'vue-router'  // 引入vue-router插件

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'

// 创建并暴露一个路由器
export default new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            path: "/about",  // 最外层路由一定需要添加/前缀
            component: About
        },
        {
            path: "/home",
            component: Home
        }
    ]
})
~~~

### src/pages/About.vue 自定义的路由组件About
~~~vue
<template>
  <h2>我是About的内容</h2>
</template>

<script>
export default {
  name: "About",
  mounted() {
    console.log("About组件挂载完毕了！");
  },
  beforeDestroy() {
    console.log("About组件即将被销毁了！");
  },
};
</script>
~~~

### src/pages/Home.vue 自定义的路由组件Home
~~~vue
<template>
  <h2>我是Home的内容</h2>
  <input type="text">
</template>

<script>
export default {
  name: "Home",
  mounted() {
    console.log("Home组件挂载完毕了！");
  },
  beforeDestroy() {
    console.log("Home组件即将被销毁了！");
  },
};
</script>
~~~

### src/App.vue App组件引入路由组件About和Home
~~~vue
<template>
  <div>
    <div class="page-header"><h2>Vue Router Demo</h2></div>

    <!-- 原始html中我们使用a标签实现页面的跳转 -->
    <!-- <a class="list-group-item active" href="./about.html">About</a> -->
    <!-- <a class="list-group-item" href="./home.html">Home</a> -->

    <!-- Vue中借助router-link标签实现路由的切换，最终router-link标签在浏览器中是a标签 -->
    <!-- 每一次切换组件后，原来的组件会被销毁而新的组件被创建 -->
    <!-- router-link的push属性设置当前页面被浏览器记录（默认），而replace属性是设置当前页面及其子路由页面不被浏览器记录，
        即跳转到其他页面后无法回退到之前的页面，但是会继续向更早的页面跳转 -->
    <router-link to="/about">About</router-link>
    <br />
    <router-link to="/home">Home</router-link>

    <div>
      <!-- keep-alive标签是让包裹设置的同一级的路由组件在切换时进行缓存从而不被销毁，做到用户的输入不被清理掉
      默认是包裹的全部路由组件不被销毁，
      不过有些情况下需要组件不被销毁但是组件内部的资源（例如定时器）销毁请在路由组件中编写deactivated钩子销毁资源（清楚定时器）
      另外路由组件中的activated钩子可以设置进入路由组件时创建资源（例如创建定时器）
      include属性是用于指定特定的路由组件不被销毁而其他的路由组件都被销毁(内部的值是组件名称，而不是路由名称)
      :include="['About','Home']" 设置多个路由组件缓存
       -->
      <!-- <keep-alive include="Home">
        <router-view></router-view>
      </keep-alive> -->

      <router-view></router-view>
    </div>

    
  </div>
</template>

<script>
export default {
  name: "App",
};
</script>
~~~






