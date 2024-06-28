# vue路由父组件传递参数给路由子组件（当然这里是指的前端数据传递，如果要到后端需要继续手动发起前端请求）
- 直接传递query参数数据：其实是?后的键值对数据
- 直接传递params参数数据：其实是路径占位参数
- 通过props传递query和params参数


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

## 父组件传递query参数数据给路由子组件

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
                    path: "detail",   // 注意子路由是不需要添加/前缀
                    component: Detail,
                }
            ]
        }
    ]
})
~~~

### src/pages/App.vue App组件引入路由组件Message
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

### src/pages/Message.vue Message组件向子路由组件Detail传递query参数数据
~~~vue
<template>
  <div>
    <ul>
      <li v-for="message in messageList" :key="message.id">
        <!-- 跳转路由并携带query参数，to的字符串写法，需要使用反引号包裹并且js变量需要使用${}包裹 -->
        <!-- <router-link :to="`/message/detail?id=${message.id}&title=${message.title}`">{{message.title}}</router-link> -->

        <!-- 跳转路由并携带query参数，to的对象写法，注意的是path的属性值必须使用单引号包裹，
        或者使用路由名称name: detail指定路由替换掉path属性（避免path路径过长） -->
        <router-link
          :to="{
            name: 'detail',
            query: {
              id: message.id,
              title: message.title,
            },
          }"
          >{{ message.title }}</router-link>
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
};
</script>
~~~

### src/pages/Detail.vue Detail组件从父路由组件Message获取传入的query参数数据
~~~vue
<template>
  <ul>
    <!-- 从路由中获取携带的query参数 -->
    <li>消息编号:{{$route.query.id}}</li>
    <li>消息标题:{{$route.query.title}}</li>
  </ul>
</template>

<script>
export default {
name:"Detail",
}
</script>
~~~
---

## 父组件传递params参数数据给路由子组件

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

### src/router/index.js 自定义的路由规则和设置传递params参数数据
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
                    path: "detail/:id/:title",   // 注意子路由是不需要添加/前缀
                    component: Detail,
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

### src/pages/Message.vue Message组件向子路由组件Detail传递params参数数据
~~~vue
<template>
  <div>
    <ul>
      <li v-for="message in messageList" :key="message.id">
        <!-- 跳转路由并携带params路径参数，to的字符串写法，需要使用反引号包裹并且js变量需要使用${}包裹 -->
        <!-- <router-link :to="`/home/message/detail/${message.id}/${message.title}`">{{message.title}}</router-link> -->

        <!-- 跳转路由并携带params路径参数，to的对象写法，注意的是传递params参数不能使用path的属性值
        只能使用路由名称name: 'detail'指定路由并且路由名称只能使用单引号包裹 -->
        <router-link
          :to="{
            name: 'detail',
            params: {
              id: message.id,
              title: message.title,
            },
          }"
          >{{ message.title }}</router-link>
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
};
</script>
~~~

### src/pages/Detail.vue Detail组件从父路由组件Message获取传入的params参数数据
~~~vue
<template>
  <ul>
    <!-- 从路由中获取携带的params参数 -->
    <li>消息编号:{{$route.params.id}}</li>
    <li>消息标题:{{$route.params.title}}</li>
  </ul>
</template>

<script>
export default {
name:"Detail",
}
</script>
~~~
---

## 父组件通过props传递query和params参数和数据给路由子组件
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

### src/router/index.js 自定义的路由规则和通过props设置传递query和params参数
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

                    // props的第一种写法，值为对象
                    // 该对象中的所有key-value都会以props的形式传递给对应的路由组件（这里是Detail组件）
                    // 在对应的路由组件中使用props进行接收参数（和父组件给子组件传递数据时子组件使用props接收数据一样）
                    // 第一种写法的缺点是传递的key-value是固定的常量，无法变化
                    // props:{id:"333",title:"你好啊！"}

                    // props的第二种写法，值为布尔，若布尔值为真，
                    // 就会把该路由组件收到的所有params参数，以props的形式传给对应的路由组件（这里是Detail组件）
                    // 第二种写法的缺点是只能传递所有params参数，无法传递query参数
                    // props: true

                    // props的第三种写法，值为函数，把返回值以props的形式传给对应的路由组件（这里是Detail组件）
                    // 优点是即可以传递params参数又可以传递query参数
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

### src/pages/Message.vue Message组件向子路由组件Detail传递query和params参数
~~~vue
<template>
  <div>
    <ul>
      <li v-for="message in messageList" :key="message.id">
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
};
</script>
~~~

### src/pages/Detail.vue Detail组件从父路由组件Message获取传入的query和params参数
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
}
</script>
~~~
---




