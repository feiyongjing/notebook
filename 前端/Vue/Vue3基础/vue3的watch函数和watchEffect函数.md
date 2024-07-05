# vue3的watch函数和watchEffect函数（用于监视属性）

### src/components/Demo.vue  
~~~vue
<template>
  <h1>{{ sum }}</h1>
  <button @click="sum++">点我加1</button>
  <hr />
  <h1>{{ msg }}</h1>
  <button @click="msg += '!'">{{ msg }}</button>
  <hr />
  <h1>姓名：{{ person.name }}</h1>
  <h1>年龄：{{ person.age }}</h1>
  <h1>薪资：{{ person.job.j1.salary }}K</h1>
  <button @click="person.name += '!'">修改姓名</button>
  <button @click="person.age++">年龄增长</button>
  <button @click="person.job.j1.salary++">涨薪</button>
</template>

<script>
// 引入watch函数和watchEffect函数监视属性
import { ref, reactive, watch,watchEffect } from "vue";

export default {
  name: "Demo",
  components: {},

  // vue2中的data、methods、等都写在setup中
  setup() {
    let sum = ref(0);
    let msg = ref("你好啊");
    let person = reactive({
      name: "张三",
      age: 18,
      job: {
        j1: {
          salary: 20,
        },
      },
    });

    // 情况1：监视ref所定义的一个响应式数据
    // watch传递的第一个参数表示需要监视的属性，如果监视ref基本数据类型直接写属性名，如果监视ref对象类型请使用 对象名.value
    // watch传递的第二个参数是一个函数，函数接收的参数是新的值和旧的值
    // watch传递的第三个参数是一个对象，可以设置immediate:true 是否默认第一次显示就监视
    // watch(sum,(newValue,oldValue)=>{
    //   console.log("sum的值变了，新的值和旧的值分别是",newValue,oldValue)
    // },{immediate:true})

    // 情况2：监视ref所定义的多个响应式数据
    // watch传递的第一个参数是一个数组表示需要监视的属性，如果监视ref基本数据类型直接写属性名，如果监视ref对象类型请使用 对象名.value
    // watch传递的第二个参数是一个函数，函数接收的参数是两个数组
    // 两个数组分别是新的值数组和旧的值数组，并且按照监视的属性数组索引对应每个数组索引中的元素
    // watch传递的第三个参数是一个对象，可以设置immediate:true 是否默认第一次显示就监视
    // watch([sum, msg], (newValue, oldValue) => {
    //   console.log("sum或msg的值变了，新的值和旧的值分别是", newValue, oldValue);
    // },{immediate:true});

    // 情况三：监视reactive所定义的一个响应式数据的全部属性
    // 1注意：监视对象的全部属性时无法正确的获取oldValue
    // 2注意：reactive强制开启了深度监视，watch传递的第三个参数中对象的deep属性（深度监视设置无效）无需设置
    // watch(person, (newValue, oldValue) => {
    //   console.log("person中的属性数据变化了，新的值和旧的值分别是", newValue, oldValue);
    // },{deep:false}); 
    // 此处的deep设置无效

    // 情况四：监视reactive所定义的一个响应式数据的某一个属性
    // watch传递的第一个参数是一个函数返回表示需要监视的属性
    // watch(()=>person.age, (newValue, oldValue) => {
    //   console.log("person中的age属性数据变化了，新的值和旧的值分别是", newValue, oldValue);
    // });

    // 情况五：监视reactive所定义的一个响应式数据的多个指定属性
    // watch传递的第一个参数是一个数组，数组的每个元素都是函数，函数返回表示需要监视的属性
    // watch([()=>person.name,()=>person.age], (newValue, oldValue) => {
    //   console.log("person中的name或age属性数据变化了，新的值和旧的值分别是", newValue, oldValue);
    // });


    // 特殊情况五：监视reactive所定义的对象中的某个属性，所以deep配置有效
    // watch(()=>person.job, (newValue, oldValue) => {
    //   console.log("person中对象的salary属性数据变化了，新的值和旧的值分别是", newValue, oldValue);
    // },{deep:true}); 此处的deep设置有效


    // watchEffect监视属性
    // 直接写监视的回调函数，回调函数中用到的属性都会监视，即当回调函数中使用的属性变化时回调函数就会执行
    watchEffect(()=>{
      const x1= sum.value
      const x2= person.job.j1.salary
      console.log("watchEffect所指定的回调执行了")
    })

    // 返回一个对象包含的data、函数、计算属性可以直接在模板中使用
    return {
      sum,
      msg,
      person,
    };
  },
};
</script>
~~~



