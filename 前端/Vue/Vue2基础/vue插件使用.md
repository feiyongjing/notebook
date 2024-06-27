# vue插件使用

## vue自定义插件使用
### src/main.js 程序启动入口设置使用插件
~~~js
import Vue from 'vue'
import App from './App'
import {myPlugin} from './js/myPlugin.js'  // 引用自定义插件

// 关闭Vue的生产提示
Vue.config.productionTip = false

// 应用插件
Vue.use(myPlugin)

const vm=new Vue({
  render: h => h(App),
}).$mount('#app')
~~~

### src/js/myPlugin.js 自定义的插件
~~~js
import dayjs from './dayjs.min.js'  // 需要提前下载 https://cdn.bootcdn.net/ajax/libs/dayjs/1.11.11/dayjs.min.js 到指定位置

export const myPlugin = {
    // 插件可以设置多个过滤器、多个自定义组件、多个mixin混入复用代码、多个Vue的原型属性
    install(Vue) {

        // 设置过滤器
        Vue.filter('timeFormater', function (value, str = "YYYY-MM-DD HH:mm:ss") {
            return dayjs(value).format(str)
        })

        // 设置自定义的指令
        Vue.directive("fbind", {
            // 自定义的指令与标签绑定成功时，注意这个时候的element是虚拟DOM，这个时候不是真实DOM无法获取它的焦点
            bind(element, binding) {
                element.value = binding.value
            },
            // 指令所在元素被插入页面时
            inserted(element, binding) {
                element.focus()
            },
            // 指令所在模板被重新解析时调用
            update(element, binding) {
                element.value = binding.value
                element.focus()
            },

        })

        // 设置mixin混入复用代码
        Vue.mixin({
            data() {
                return {
                  name: "育才小学",
                };
            },
        
            methods: {
                showName() {
                    alert(this.name)
                }
            },
        })

        // 向Vue的原型放入属性，后续vm和vc都能正常使用
        Vue.prototype.hello=()=>{alert("你好啊")}
    }
}
~~~

### src/App.vue App组件管理其他组件
~~~vue
<template>
  <div>
    <div class="row">
      <School></School>
      <Student></Student>
    </div>
  </div>
</template>

<script>
import School from "./components/School.vue";
import Student from "./components/Student.vue";

export default {
  name: "App",
  components: {
    School,
    Student,
  },
};
</script>

<style scoped>
</style>
~~~

### src/components/School.vue School组件
~~~vue
<template>
  <div class="school">
    <!-- 过滤器 -->
    <h3>现在的时间格式化后是：{{Date.now() | timeFormater("YYYY-MM-DD HH:mm:ss")}}</h3>

    <!-- 自定义指令 -->
    <div>当前的m值是{{m}}</div>
    <button @click="m++">点我m+1</button>
    <div>当m发生变化时以下自定义的v-fbind指令所在的input标签会自动获取标签</div>
    <input type="text" v-fbind:value="m">

    <!-- mixin混入复用代码 -->
    <h3>学校名称：{{ name }}</h3>
    <button @click="showName">点我弹出学校名称</button>

    <!-- 调用Vue原型属性 -->
    <button @click="hello">点我弹出打招呼</button>
  </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      m:0,
      name: "希望小学",
    };
  },

};
</script>

<style>
.school {
  background-color: aquamarine;
}
</style>
~~~

### src/components/Student.vue Student组件
~~~vue
<template>
  <div class="student">
    <!-- mixin混入复用代码 -->
    <h3>学生学校名称：{{ name }}</h3>
    <h3>学生年龄：{{ age }}</h3>
    <button @click="showName">点我弹出学生名称</button>
  </div>
</template>

<script>

export default {
  name: "Student",
  data() {
    return {
      age: 12,
    };
  },

};
</script>

<style>
.student {
  background-color: blueviolet;
}
</style>
~~~

## vue使用外部插件，例如外部模板样式之类的UI
~~~
~~~










