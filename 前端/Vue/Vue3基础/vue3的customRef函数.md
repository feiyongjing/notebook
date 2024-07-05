# vue3的customRef函数（自定义ref函数）

### src/App.vue 
~~~vue
<template>
  <input type="text" v-model="keyWork" />
  <h3>{{ keyWork }}</h3>
</template>

<script>
import { ref, customRef } from "vue";

export default {
  name: "App",
  components: {},

  setup() {
    // 自定义一个ref名为：myRef
    // 修改数据会延迟一秒生效
    function myRef(value,delay) {
      let timer
      
      return customRef((track, trigger) => {
        return {
          get() {
            track(); // track通知Vue追踪value的变化（vue提前和get商量一下，vue会在数据发生变化后获取get返回value）
            return value;
          },
          set(newValue) {
            clearTimeout(timer) // 根据定时器ID删除定时器防抖
            timer= setTimeout(() => {
              value = newValue; // 修改数据
              trigger(value); // 通知Vue去重新解析模板
            },delay);
          },
        };
      });
    }
    // let keyWork = ref("hello")
    let keyWork = myRef("hello",1000);
    return {
      keyWork,
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

