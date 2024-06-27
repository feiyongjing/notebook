# vue组件模板的style样式设置
### src/main.js 程序启动入口
~~~js
import Vue from 'vue'
import App from './App'

// 关闭Vue的生产提示
Vue.config.productionTip = false

const vm=new Vue({
  render: h => h(App),
}).$mount('#app')
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
  <div class="demo">
    <h3>学校名称：{{ name }}</h3>
    <button @click="showName">点我弹出学校名称</button>
  </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      name: "希望小学",
    };
  },

  methods: {
    showName() {
      alert(this.name);
    },
  },
};
</script>

<style scoped>
/* style标签中可以写scoped使该css只应用于当前组件模板
   lang属性可以设置特殊的css，默认是正常css，也可以写less
   如果模板没有使用样式，则style标签可以省略不写
 */
.demo {
  background-color: aquamarine;
}
</style>
~~~

### src/components/Student.vue Student组件
~~~vue
<template>
  <div class="demo">
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
      name: "张三",
      age: 12,
    };
  },

  methods: {
    showName() {
      alert(this.name);
    },
  },
};
</script>

<style scped>
/* style标签中可以写scoped使该css只应用于当前组件模板
   lang属性可以设置特殊的css，默认是正常css，也可以写less
   如果模板没有使用样式，则style标签可以省略不写
 */
.demo {
  background-color: blueviolet;
}
</style>
~~~


