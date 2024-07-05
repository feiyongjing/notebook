# vue3的响应式数据的判断函数

### src/App.vue 
~~~vue
<template>
  <div class="app">
    <h3>我是App组件</h3>
  </div>
</template>

<script>
import {
  ref,
  reactive,
  readonly,
  isRef,
  isReactive,
  isReadonly,
  isProxy,
} from "vue";

export default {
  name: "App",

  setup() {
    let sum = ref(0);
    let car = reactive({
      name: "奔驰",
      price: "40w",
    });
    let car2 = readonly(car);

    console.log("判断是否是ref对象", isRef(sum));
    console.log("判断是否是reactive创建的响应式代理", isReactive(car));
    console.log("判断是否是readonly创建只读代理", isReadonly(car2));
    console.log("判断reactive创建的响应式对象是否是代理", isProxy(car));
    console.log("判断readonly创建的对象是否是代理", isProxy(car2));

    return {};
  },
};
</script>

<style>
.app {
  background-color: grey;
  padding: 10px;
}
</style>
~~~


