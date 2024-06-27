# mixins混入复用公共代码
### App父组件
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

### School和Student子组件复用的代码showNameMixin.js
~~~js
export const showNameMixin = {
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

    mounted() {
        console.log("mounted1")
    },
}
~~~

### School子组件
~~~vue
<template>
  <div class="school">
    <h3>学校名称：{{ name }}</h3>
    <button @click="showName">点我弹出学校名称</button>
  </div>
</template>

<script>
import { showNameMixin } from "./showNameMixin.js";   // 引入复用的代码

export default {
  name: "School",
  data() {
    return {
      name: "希望小学",
    };
  },

  // 挂载钩子函数
  mounted() {
      console.log("mounted2")
  },

  // 通过mixins设置需要复用代码
  // 复用的代码中data属性、methods的函数、mounted挂载钩子函数等会直接复用
  // 注意如果一些属性或函数在vc组件中重复编写了，则以vc组件为主，但是mounted挂载钩子函数是都生效的
  mixins: [showNameMixin],
};
</script>

<style>
.school {
  background-color: aquamarine;
}
</style>
~~~

### Student子组件
~~~vue
<template>
  <div class="student">
    <h3>学生名称：{{ name }}</h3>
    <h3>学生年龄：{{ age }}</h3>
    <button @click="showName">点我弹出学生名称</button>
  </div>
</template>

<script>
import { showNameMixin } from "./showNameMixin.js";  // 引入复用的代码

export default {
  name: "Student",
  data() {
    return {
      name: "张三",
      age: 12,
    };
  },

  // 通过mixins设置需要复用代码
  // 复用的代码中data属性、methods的函数、mounted挂载钩子函数等会直接复用
  // 注意如果一些属性或函数在vc组件中重复编写了，则以vc组件为主，但是mounted挂载钩子函数是都生效的
  mixins: [showNameMixin],
};
</script>

<style>
.student {
  background-color: blueviolet;
}
</style>
~~~





