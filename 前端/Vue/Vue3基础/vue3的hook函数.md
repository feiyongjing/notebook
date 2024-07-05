# vue3的hook函数（类似于vue2的mixins复用代码）

## 项目目录如下
├── src
│   ├── components    放一般组件
│   │   └── Demo.vue
│   ├── hook            hook函数将要复用的js代码
│   │   └── usePoint.js
│   ├── App.vue       
│   └── main.js


### src/hook/usePoint.js 
~~~js
import { reactive, onMounted, onBeforeUnmount } from "vue";

export default function () {
    let point = reactive({
        x: 0,
        y: 0,
    });

    function savePoint(event) {
        point.x = event.pageX;
        point.y = event.pageY;
        console.log(event.pageX, event.pageY)
    }

    onMounted(() => {
        window.addEventListener("click", savePoint);
    });

    onBeforeUnmount(() => {
        window.removeEventListener("click", savePoint);
    });

    return point
}
~~~


### src/components/Demo.vue 
~~~vue
<template>
  <h1>{{ sum }}</h1>
  <button @click="sum++">点我加1</button>
  <hr />
  <h2>当前点击时鼠标的坐标为：x：{{ point.x }}，y：{{ point.y }}</h2>
</template>

<script>
// 在setup在使用组合式API写生命周期，
import { ref, reactive, onMounted, onBeforeUnmount } from "vue";

// 引入自定义的hook函数，本质上是把setup中使用的Composition API进行了封装，使得多个组件都可以复用代码
import usePoint from "../hooks/usePoint";

export default {
  name: "Demo",
  components: {},

  setup() {
    let sum = ref(0);

    // 获取点位数据，至于点位数据的维护由引入hook代码内部维护
    let point=usePoint()
    
    // 返回一个对象包含的data、函数、计算属性可以直接在模板中使用
    return {
      sum,
      point,
    };
  },
};
</script>
~~~

