# vue3的toRaw函数和markRaw函数（用于去除对象的响应式）

### src/components/Demo.vue
~~~vue
<template>
  <h1>a的值是{{ sum.a }}</h1>
  <button @click="sum.a++">点我a加一</button>
  <h1>姓名：{{ person.name }}</h1>
  <h1>年龄：{{ person.age }}</h1>
  <h1>薪资：{{ person.job.j1.salary }}K</h1>

  <h1 v-show="person.car">座驾：{{ person.car }}</h1>
  <button @click="person.name += '!'">修改姓名</button>
  <button @click="person.age++">年龄增长</button>
  <button @click="person.job.j1.salary++">涨薪</button>
  <button @click="showRawPerson">显示原始的person</button>
  <button @click="addCar">给人添加一台车</button>

  <button @click="person.car.name += '!'">修改车名</button>
  <button @click="person.car.price++">修改车价格</button>
</template>

<script>
// toRaw函数将一个响应式的对象变成一个普通的对象
// markRaw函数标记一个的对象该对象永远都是一个普通的对象，不会再成为响应式的对象
import { ref, reactive, toRaw, markRaw } from "vue";

export default {
  name: "Demo",
  components: {},

  setup() {
    let sum = ref({ a: 0 });

    let person = reactive({
      name: "张三",
      age: 18,
      job: {
        j1: {
          salary: 20,
        },
      },
    });

    function showRawPerson() {
      // 输出的是Proxy对象
      // console.log("显示person响应式对象", person);

      // 输出的是一个Object对象，而不是一个Proxy对象
      // toRaw只能处理reactive缔造的响应式对象，ref缔造的响应式对象由于包裹在RefImpl中无法处理
      console.log("显示原始的person对象", toRaw(person));
    }

    function addCar() {
      let car = { name: "奔驰", price: 40 };

      // 向person上添加一个属性对象值该属性也默认是响应式的
      // person.car = car;

      // 向person上添加一个属性对象值，
      // 但是这个对象使用markRaw标记了，函数返回的对象永远都不是响应式的对象
      person.car = markRaw(car);

    }

    return {
      sum,
      person,
      showRawPerson,
      addCar,
    };
  },
};
</script>
~~~


