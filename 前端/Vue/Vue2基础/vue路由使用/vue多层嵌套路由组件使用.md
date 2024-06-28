# vue多层嵌套路由组件使用

### 项目目录如下
- src/router目录下放路由组件，src/components目录下放一般组件（当然当前案例没有使用一般组件）
~~~shell
/
└── src
    ├── components
    │   └── xxx.vue
    ├── pages
    │   ├── About.vue 
    │   ├── Home.vue 
    │   ├── Message.vue 
    │   └── News.vue 
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

### src/router/index.js 自定义的路由规则和嵌套的子路由（路由器）
~~~js
// 引入vue-router插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'
import Message from '../pages/Message'
import News from '../pages/News'

// 创建并暴露一个路由器
export default new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            path: "/about",  // 最外层路由一定需要添加/前缀
            component: About
        },
        {
            path: "/home",   // 最外层路由一定需要添加/前缀
            component: Home,
            // 设置子路由，路由到不同的组件
            children:[
                {
                    path:"message",   // 注意子路由是不需要添加/前缀
                    component: Message
                },
                {
                    name: 'news',   // 给路由命名后在router-link标签的to属性值对象写法，从而简化过长的path
                    component:News
                }
            ]
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
};
</script>
~~~

### src/pages/Home.vue 自定义的路由组件Home，该组件会继续嵌套路由组件
~~~vue
<template>
  <div>
    <h2>我是Home的内容</h2>
    <div>
      <ul class="nav nav-tabs">
        <li>
          <!-- 注意to跳转的路由路径需要写全路径-->
          <router-link
            to="/home/message"   
            >message</router-link
          >
        </li>
        <li>
          <!-- to属性值对象写法可以写设置好的name，从而简化过长的path  -->
          <router-link
            :to="{
              name: 'news'
            }"
            >news</router-link
          >
        </li>
      </ul>
      <!-- 指定组件的呈现位置 -->
      <router-view></router-view>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
};
</script>
~~~

### src/pages/Message.vue 自定义的路由组件Message，该组件会被Home路由组件嵌套 
~~~vue
<template>
  <div>
    <ul>
      <li><a href="/message1">messages001</a>&nbsp;&nbsp;</li>
      <li><a href="/message2">messages002</a>&nbsp;&nbsp;</li>
      <li><a href="/message3">messages003</a>&nbsp;&nbsp;</li>
    </ul>
  </div>
</template>

<script>
export default {
    name:"Message",
};
</script>
~~~

### src/pages/News.vue 自定义的路由组件News，该组件会被Home路由组件嵌套 
~~~vue
<template>
  <div>
    <ul>
      <li>news001</li>
      <li>news002</li>
      <li>news003</li>
    </ul>
  </div>
</template>

<script>
export default {
  name:"News",
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
    <router-link to="/about">About</router-link>
    <br />
    <router-link to="/home">Home</router-link>

    <div>
      <!-- 指定路由组件的呈现位置 -->
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




