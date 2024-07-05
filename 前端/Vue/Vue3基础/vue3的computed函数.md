# vue3的computed函数（用于计算属性）

### src/components/Demo.vue 
~~~vue
<template>
  <!-- vue3组中的模板可以没有根标签 -->
  <h1>一个人的信息</h1>
  姓：<input type="text" v-model="person.firstName" /> <br />
  名:<input type="text" v-model="person.lastName" /><br />
  <h2>姓名：{{ person.fullName }}</h2>
  姓名:<input type="text" v-model="person.fullName" /><br />
</template>

<script>
// 引入computed计算属性
import { ref, reactive, computed } from "vue";

export default {
  name: "Demo",
  components: {},

  // vue2中的data、methods、等都写在setup中
  setup() {
    let person = reactive({
      firstName: "张",
      lastName: "三",
    });

    // 计算属性使用computed函数，
    // 简写方式（计算属性只考虑读取的情况使用）：参数是一个函数返回值是计算属性的结果
    // person.fullName = computed(() => person.firstName + person.lastName);

    // 完整写法方式（计算属性只考虑读取和写的情况使用）：参数是一个对象
    person.fullName = computed({
      get() {
        return person.firstName + "-" + person.lastName;
      },
      set(value) {
        const nameArr = value.slice("-");
        person.firstName = nameArr[0];
        person.lastName = nameArr[1];
      },
    });

    // 返回一个对象包含的data、函数、计算属性可以直接在模板中使用
    return {
      person,
    };
  },
};
</script>
~~~


