# vue不使用router-link标签进行路由

### 几个传递参数的项目目录都是一样的，如下
- src/router目录下放路由组件，src/components目录下放一般组件（当然当前案例没有使用一般组件）
~~~shell
/
└── src
    ├── components
    │   └── xxx.vue
    ├── pages
    │   ├── Detail.vue 
    │   └── Message.vue 
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

### src/router/index.js 自定义的路由规则
~~~js
// 引入VueRouter插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import Message from '../pages/Message'
import Detail from '../pages/Detail'


// 创建并暴露一个路由器
export default new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            path: "/message",   // 注意子路由是不需要添加/前缀
            component: Message,
            children: [
                {
                    name: 'detail',   // 给路由命名后在to的对象写法中可以使用name简化过长的path
                    path: "detail/:id/",   // 注意子路由是不需要添加/前缀
                    component: Detail,

                    props($route) {
                        return {
                            id: $route.params.id,      // params参数传递
                            title: $route.query.title  // query参数传递
                        }
                    }
                }
            ]

        }
    ]
})
~~~

### src/App.vue App组件引入路由组件Message
~~~vue
<template>
  <div>
    <div class="page-header"><h2>Vue Router Demo</h2></div>

    <router-link
      :to="{
        path: '/message',
      }"
      >Message</router-link
    >
    
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

### src/pages/Message.vue Message路由组件 设置通过按钮回调实现路由跳转
~~~vue
<template>
  <div>
    <ul>
      <li v-for="message in messageList" :key="message.id">
        <!-- router-link的push属性设置当前页面被浏览器记录，而replace属性是设置当前页面及其子路由页面不被浏览器记录，
        即跳转到其他页面后无法回退到之前的页面，但是会继续向更早的页面跳转 -->
        <router-link 
          :to="{
            name: 'detail',
            params: {
              id: message.id,
            },
            query: {
              title: message.title,
            },
          }"
          >{{ message.title }}</router-link>

          <!-- 不使用router-link标签实现路由跳转 -->
          <button @click="pushShow(message)" >push跳转</button>
          <button @click="replaceShow(message)">replace跳转</button>
      </li>
    </ul>

    <!-- 指定组件的呈现位置 -->
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: "Message",
  data() {
    return {
      messageList: [
        { id: "001", title: "消息001" },
        { id: "002", title: "消息002" },
        { id: "003", title: "消息003" },
      ],
    };
  },

  methods:{
    pushShow(message){
      // this.$router.push，进行路由的跳转，并且当前页面被浏览器记录，可以回退到当前页面
      // 参数和router-link标签的to属性对象写法一样
      this.$router.push({
            name: 'detail',
            params: {
              id: message.id,
            },
            query: {
              title: message.title,
            },
          });
    },
    replaceShow(message){
      // this.$router.replace进行路由的跳转，而当前页面不被浏览器记录，不可以回退到当前页面
      // 参数和router-link标签的to属性对象写法一样
      this.$router.replace({
            name: 'detail',
            params: {
              id: message.id,
            },
            query: {
              title: message.title,
            },
          });
    }
  }
};
</script>
~~~

### src/pages/Detail.vue Detail路由组件
~~~vue
<template>
  <ul>
    <!-- 如果路由有设置props时，通过props获取路由传递的参数直接使用 -->
    <li>消息编号:{{id}}</li>
    <li>消息标题:{{title}}</li>
  </ul>
</template>

<script>
export default {
name:"Detail",

props:['id','title']

}
</script>
~~~












