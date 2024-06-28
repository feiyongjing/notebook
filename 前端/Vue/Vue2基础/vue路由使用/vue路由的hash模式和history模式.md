# vue路由的hash模式和history模式

### src/router/index.js 自定义的路由规则设置路由模式
~~~js
// 引入VueRouter插件
import VueRouter from 'vue-router'

// 引入路由需要的路由组件，路由组件一般放在pages目录中，而一般组件放在components目录中
import About from '../pages/About'
import Home from '../pages/Home'


// 创建并暴露一个路由器
const router = new VueRouter({

    // mode默认是hash模式，会在域名后的路径添加/#/ 会将/#/后面的路径当成路由路径去获取前端资源
    // history模式不会在域名后的路径添加/#/ 注意路由的路径页面刷新后直接拿着这个路径去获取后端资源而不是前端资源导致报错
    // history模式的解决方案是通过nginx来区分前端路由和后端路由进行代理转发
    // 一般如果前后端API都是同一个域名是使用的 history 模式
    mode: "hash",

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


