# vue的路由守卫
- 全局路由守卫
  - 全局前置路由守卫
  - 全局后置路由守卫
- 独享前置路由守卫（独享路由没有后置路由守）
- 组件内路由守卫

## 几个路由守卫的项目目录都是一样的，如下
- src/router目录下放路由组件，src/components目录下放一般组件
~~~shell
/
└── src
    ├── components
    │   └── Banner.vue
    ├── pages
    │   ├── About.vue 
    │   └── Home.vue 
    ├── router
    │   └── index.js
    ├── App.vue
    └── main.js
~~~

## 全局路由守卫（包含前置守卫和后置守卫）
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

### src/router/index.js 自定义的路由规则并且设置全局前置路由守卫和全局后置路由守卫
~~~js
// 引入VueRouter插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'


// 创建并暴露一个路由器
const router = new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            name: 'about',
            path: "/about",
            component: About,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "关于",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: true,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            }
        },
        {
            name: 'home',
            path: "/home",
            component: Home,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "首页",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: false,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            },
        }
    ]
})

// 全局前置路由守卫：设置初始化或者是在每次路由切换之前被调用的函数，3个参数默认有不同的用处
// to 参数是切换路由时路由的终点路由
// from 参数是切换路由时路由的起点路由
// next 参数是一个函数变量，被调用就跳转到下一个路由，不被调用则路由被拦截下来了
router.beforeEach((to, from, next) => {
    console.log("路由去向", to)
    console.log("来源路由", from)

    // 检查是否有权限查看页面
    if (to.meta.isAuth) {
        // 检查浏览器的localStorage是否包含指定数据
        if (localStorage.getItem('school') === '育才小学') {
            next()
        } else {
            alert("localStorage中必须有school: 育才小学 才能跳转路由")
        }
    } else {
        next()
    }
})


// 全局后置路由守卫：设置初始化或者是在每次路由切换之后被调用的函数，2个参数默认有不同的用处
// to 参数是切换路由时路由的终点路由
// from 参数是切换路由时路由的起点路由
router.afterEach((to, from) => {
    console.log("路由去向", to)
    console.log("来源路由", from)

    // 获取去向路由的meta元数据中获取数据替换页签文字，如果没有就是默认的透明系统
    document.title=to.meta.title || "透明系统"
})

export default router 
~~~

### src/App.vue App组件管理和引入其他组件
~~~vue
<template>
  <div>
    <Banner />
    <router-link to="/about">About</router-link>
    <br/>
    <router-link to="/home">Home</router-link>
    <!-- 指定组件的呈现位置 -->
    <router-view></router-view>
  </div>
</template>

<script>
import Banner from "./components/Banner.vue";

export default {
  name: "App",
  components: {
    Banner,
  },
};
</script>
~~~

### src/components/Banner.vue 一般组件Banner，内部设置了路由组件的跳转
~~~vue
<template>
  <div >
    <h2>Vue Router Demo</h2>
    <button @click="back">回退</button>
    <button @click="froward">前进</button>
    <button @click="go">测试一下go</button>
  </div>
</template>

<script>
export default {
  name: "Banner",

  methods: {
    back() {
      // 页面回退到之前的页面
      this.$router.back();
    },
    froward() {
      // 页面前进到之前的页面
      this.$router.forward();
    },
    go(){
      // 页面前进或后退，传入的参数是正数就前进几次到之前的页面，负数就后退几次到之前的页面
      this.$router.go(-1);
    }
  },
};
</script>
~~~

### src/pages/About.vue About路由组件
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

### src/pages/Home.vue Home路由组件
~~~vue
<template>
  <div>
    <h2>我是Home的内容</h2>
  </div>
</template>

<script>
export default {
  name: "Home",
};
</script>
~~~
---



## 独享前置路由守卫（独享路由没有后置路由守）
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

### src/router/index.js 自定义的路由规则并且设置单个路由组件的路由守卫
~~~js
// 引入VueRouter插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'


// 创建并暴露一个路由器
const router = new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            name: 'about',
            path: "/about",
            component: About,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "关于",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: true,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            },

            // 独享前置路由守卫：设置初始化或者是在每次切换到该路由时被调用的函数，3个参数默认有不同的用处
            // to 参数是切换路由时路由的终点路由
            // from 参数是切换路由时路由的起点路由
            // next 参数是一个函数变量，被调用就跳转到下一个路由，不被调用则路由被拦截下来了
            // 注意独享路由没有后置路由守卫，只能使用全局后置路由守卫处理路由切换后的事情
            beforeEnter: (to, from, next) => {
                console.log("路由去向", to)
                console.log("来源路由", from)

                // 检查是否有权限查看页面
                if (to.meta.isAuth) {
                    // 检查浏览器的localStorage是否包含指定数据
                    if (localStorage.getItem('school') === '育才小学') {
                        next()
                    } else {
                        alert("localStorage中必须有school: 育才小学 才能跳转路由")
                    }
                } else {
                    next()
                }
            }
        },
        {
            name: 'home',
            path: "/home",
            component: Home,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "首页",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: false,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            },
        }
    ]
})

export default router 
~~~

### src/App.vue App组件管理和引入其他组件
~~~vue
<template>
  <div>
    <Banner />
    <router-link to="/about">About</router-link>
    <br/>
    <router-link to="/home">Home</router-link>
    <!-- 指定组件的呈现位置 -->
    <router-view></router-view>
  </div>
</template>

<script>
import Banner from "./components/Banner.vue";

export default {
  name: "App",
  components: {
    Banner,
  },
};
</script>
~~~

### src/components/Banner.vue 一般组件Banner，内部设置了路由组件的跳转
~~~vue
<template>
  <div >
    <h2>Vue Router Demo</h2>
    <button @click="back">回退</button>
    <button @click="froward">前进</button>
    <button @click="go">测试一下go</button>
  </div>
</template>

<script>
export default {
  name: "Banner",

  methods: {
    back() {
      // 页面回退到之前的页面
      this.$router.back();
    },
    froward() {
      // 页面前进到之前的页面
      this.$router.forward();
    },
    go(){
      // 页面前进或后退，传入的参数是正数就前进几次到之前的页面，负数就后退几次到之前的页面
      this.$router.go(-1);
    }
  },
};
</script>
~~~

### src/pages/About.vue About路由组件
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

### src/pages/Home.vue Home路由组件
~~~vue
<template>
  <div>
    <h2>我是Home的内容</h2>
  </div>
</template>

<script>
export default {
  name: "Home",
};
</script>
~~~
---



## 组件内路由守卫
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

### src/router/index.js 自定义的路由规则并且设置单个路由组件的路由守卫
~~~js
// 引入VueRouter插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'


// 创建并暴露一个路由器
const router = new VueRouter({
    // 设置不同路径路由到不同的组件
    routes: [
        {
            name: 'about',
            path: "/about",
            component: About,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "关于",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: true,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            },
        },
        {
            name: 'home',
            path: "/home",
            component: Home,
            // meta是用于配置一些元数据，通常这些元数据会在路由守卫中使用
            meta: {
                title: "首页",   // 目前是在路由守卫中获取title数据替换跳转路由的页签
                isAuth: false,   // 目前是在路由守卫中判断isAuth来确认是否有权限切换到该路由
            },
        }
    ]
})

export default router 
~~~

### src/App.vue App组件管理和引入其他组件
~~~vue
<template>
  <div>
    <Banner />
    <router-link to="/about">About</router-link>
    <br/>
    <router-link to="/home">Home</router-link>
    <!-- 指定组件的呈现位置 -->
    <router-view></router-view>
  </div>
</template>

<script>
import Banner from "./components/Banner.vue";

export default {
  name: "App",
  components: {
    Banner,
  },
};
</script>
~~~

### src/components/Banner.vue 一般组件Banner，内部设置了路由组件的跳转
~~~vue
<template>
  <div >
    <h2>Vue Router Demo</h2>
    <button @click="back">回退</button>
    <button @click="froward">前进</button>
    <button @click="go">测试一下go</button>
  </div>
</template>

<script>
export default {
  name: "Banner",

  methods: {
    back() {
      // 页面回退到之前的页面
      this.$router.back();
    },
    froward() {
      // 页面前进到之前的页面
      this.$router.forward();
    },
    go(){
      // 页面前进或后退，传入的参数是正数就前进几次到之前的页面，负数就后退几次到之前的页面
      this.$router.go(-1);
    }
  },
};
</script>
~~~

### src/pages/About.vue About路由组件
~~~vue
<template>
  <h2>我是About的内容</h2>
</template>

<script>
export default {
  name: "About",

  // 组件内路由守卫：通过路由规则进入该组件时被调用，3个参数默认有不同的用处
  // to 参数是切换路由时路由的终点路由
  // from 参数是切换路由时路由的起点路由
  // next 参数是一个函数变量，被调用就跳转到下一个路由，不被调用则路由被拦截下来了
  beforeRouteEnter: (to, from, next) => {
    console.log("路由去向", to);
    console.log("来源路由", from);

    // 检查是否有权限查看页面
    if (to.meta.isAuth) {
      // 检查浏览器的localStorage是否包含指定数据
      if (localStorage.getItem("school") === "育才小学") {
        next();
      } else {
        alert("localStorage中必须有school: 育才小学 才能跳转路由");
      }
    } else {
      next();
    }
  },

   // 组件内路由守卫：通过路由规则离开该组件时被调用，2个参数默认有不同的用处
   // to 参数是切换路由时路由的终点路由
   // from 参数是切换路由时路由的起点路由
   // next 参数是一个函数变量，被调用就跳转到下一个路由，不被调用则路由被拦截下来了
   beforeRouteLeave: (to, from, next) => {
    // next如果不被调用则路由不会切换到其他的路由
    // next()
    next()
   }
};
</script>
~~~

### src/pages/Home.vue Home路由组件
~~~vue
<template>
  <div>
    <h2>我是Home的内容</h2>
  </div>
</template>

<script>
export default {
  name: "Home",
};
</script>
~~~
---




