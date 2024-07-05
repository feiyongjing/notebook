# vue3的toRef函数和toRefs函数（用于获取和修改数据）

### src/components/Demo.vue 
~~~vue
<template>
  <h1>姓名：{{ name }}</h1>
  <h1>年龄：{{ age }}</h1>
  <h1>薪资：{{ job.j1.salary }}K</h1>
  <button @click="name += '!'">修改姓名</button>
  <button @click="age++">年龄增长</button>
  <button @click="job.j1.salary++">涨薪</button>
</template>

<script>

// toRef函数是用于获取对象的RefImpl对象，并且通过RefImpl对象获取和修改原始对象指定属性的数据
// toRef函数的第一参数是原始对象，第二参数是原始对象的属性名称
// toRefs函数是用于获取对象的RefImpl对象，通过RefImpl对象可以获取和修改原始对象的全部属性数据
// toRefs函数的第一参数是原始对象，没有第二个参数
import { ref, reactive, toRef, toRefs } from "vue";

export default {
  name: "Demo",
  components: {},

  setup() {
    let person = reactive({
      name: "张三",
      age: 18,
      job: {
        j1: {
          salary: 20,
        },
      },
    });

    // 返回一个对象包含的data、函数、计算属性可以直接在模板中使用
    return {
      // 第一种写法
      // person, 模板中需要使用person.name获取属性

      // 第二种写法使用toRef
      // name:toRef(person,"name"),  模板中直接使用name获取属性
      // age:toRef(person,"age"),    模板中直接使用age获取属性
      // salary: toRef(person.job.j1, "salary"), 模板中直接使用salary获取属性,

      // 第三种写法使用toRefs，直接解析对象下的属性
      // 优点是不用一个一个属性toRef
      // 缺点是对象下的属性如果是对象，但是需要是深层的数据则无法获取
      // 比如如下的person.job.j1.salary 只能简写最外面的一层，无法简写成salary，在模板在只能简写成job.j1.salary
      ...toRefs(person)  

    };
  },
};
</script>
~~~



