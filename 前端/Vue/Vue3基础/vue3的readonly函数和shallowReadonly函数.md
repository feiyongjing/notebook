# vue3的readonly函数和shallowReadonly函数（用于数据不发生修改场景）

### src/components/Demo.vue
~~~vue
<template>
  <h1>a的值是{{ sum.a }}</h1>
  <button @click="sum.a++">点我a加一</button>
  <h1>姓名：{{ person.name }}</h1>
  <h1>年龄：{{ person.age }}</h1>
  <h1>薪资：{{ person.job.j1.salary }}K</h1>
  <button @click="person.name += '!'">修改姓名</button>
  <button @click="person.age++">年龄增长</button>
  <button @click="person.job.j1.salary++">涨薪</button>
</template>

<script>
// readonly,shallowReadonly组合式API都是让数据变为只读，不允许写
// 区别：readonly是能做全部数据包含深层次的数据只读，不允许写
// 区别：shallowReadonly只能做浅层次的数据只读，不允许写
import { ref, reactive, readonly,shallowReadonly} from "vue";

export default {
  name: "Demo",
  components: {},

  setup() {

    let sum = ref({a:0});
    // readonly保护全部数据包含深层次的数据不被修改，
    // 返回深拷贝的数据，拷贝的数据无法修改
    // 所以无法修改a
    sum=readonly(sum)
   
    let person = reactive({
      name: "张三",
      age: 18,
      job: {
        j1: {
          salary: 20,
        },
      },
    });

    // shallowReadonly保护浅层次的数据不被修改
    // 返回浅拷贝的数据，拷贝的浅层数据无法修改，但是深层的数据可以被修改
    // 所以名称和年龄无法修改，而薪资可以修改
    person=shallowReadonly(person);
   

    return {
      sum,
      person
    };
  },
};
</script>
~~~


