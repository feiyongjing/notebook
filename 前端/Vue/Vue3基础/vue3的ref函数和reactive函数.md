# vue3的ref函数和reactive函数

### src/App.vue 
~~~vue
<template>
  <!-- vue3组中的模板可以没有根标签 -->
  <h1>一个人的信息</h1>
  <h2 v-if="person.name">姓名：{{ person.name }}</h2>
  <h2>年龄：{{ person.age }}</h2>
  <h2>职业：{{ person.job.type }}</h2>
  <h2>薪资：{{ person.job.salary }}</h2>
  <h2>爱好：{{ person.hobby }}</h2>

  <h2 v-if="person.sex">性别：{{ person.sex }}</h2>

  <button @click="changeInfo">修改信息</button>

  <button @click="delName">删除名称</button>
  <button @click="addSex">添加性别</button>
</template>

<script>
// 如果数据需要响应式的变化，即数据只要发生变化那么模板的数据也随之变化
// 引入ref函数和reactive函数定义数据实现响应式，它们都会创建数据的RefImpl对象，后续操作RefImpl对象会影响数据

// ref函数可以让基本类型数据和对象类型数据都做到响应式，但是其实ref函数对对象类型数据的操作还是借助的reactive函数
// ref使用如下，ref的缺点是获取数据必须通过 .value 获取，并且在setup的返回值中大量声明数据名称
// 基本类型数据定义  let name = ref("张三");
// 获取基本类型数据  name.value
// 对象类型数据定义  let job = ref({type:'前端工程师',salary:'30k'})
// 获取对象类型数据  job.value.type

// reactive函数只能让对象类型数据做到响应式
// 数据定义  let job = reactive({type:'前端工程师',salary:'30k'})
// 获取数据  job.type

// vue3的响应式实际上是使用创建Proxy对象来代理数据对象，
// 做到数据的属性发生任意的变化（包含读取）时拦截并使用Reflect(反射)对被代理的对象属性进行操作
import { ref, reactive } from "vue";

export default {
  name: "App",
  components: {},

  // vue2中的data、methods、等都写在setup中
  setup() {
    // reactive函数直接定义对象类型变量数据
    let person = reactive({
      name: "张三",
      age: "18",
      job: {
        type: "前端工程师",
        salary: "30k",
      },
      hobby: ["抽烟", "喝酒", "烫头"],
    });

    function changeInfo() {
      person.name = "李四";
      person.age = 20;
      person.job.type = "UI工程师";
      person.job.salary = "60k";
      // 在vue3中可以直接通过数组索引修改数组数据，这是vue2无法做到的
      person.hobby[0] = "游戏";
    }

    function delName() {
      // 删除属性
      delete person.name;
    }

    function addSex() {
      // 添加属性
      person.sex = "男";
    }

    // 返回一个对象包含的data和函数可以直接在模板中使用
    return {
      person,
      changeInfo,
      delName,
      addSex,
    };
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
~~~



