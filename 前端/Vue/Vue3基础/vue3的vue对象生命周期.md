# vue3的vue对象生命周期

### src/components/Demo.vue 
~~~vue
<template>
  <h1>{{ sum }}</h1>
  <button @click="sum++">点我加1</button>
</template>

<script>
// 在setup在使用组合式API编写生命周期
import {
  ref,
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from "vue";

export default {
  name: "Demo",
  components: {},

  setup() {
    let sum = ref(0);

    console.log("由于setup会在beforeCreate和created之前执行，所以");
    console.log("beforeCreate被setup替代了");
    console.log("created被setup替代了");

    onBeforeMount(() => {
      console.log("onBeforeMount生命周期");
    });
    onMounted(() => {
      console.log("onMounted生命周期");
    });
    onBeforeUpdate(() => {
      console.log("onBeforeUpdate生命周期");
    });
    onUpdated(() => {
      console.log("onUpdated生命周期");
    });
    onBeforeUnmount(() => {
      console.log("onBeforeUnmount生命周期");
    });
    onUnmounted(() => {
      console.log("onUnmounted生命周期");
    });

    // 返回一个对象包含的data、函数、计算属性可以直接在模板中使用
    return {
      sum,
    };
  },

  // 以下是在vue3中的按照vue2的写法写生命周期，与vue2相比最后两个生命周期的名称有变化，但是功能是一致的

  // // 数据代理还未开始，无法通过Vm获取data中的数据，methods中的方法
  //   beforeCreate() {
  //       console.log("beforeCreate")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 数据监测和数据代理之后，可以通过Vm获取data中的数据，methods中的方法
  //   created() {
  //       console.log("created")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 已经生成了虚拟DOM，但是在beforeMount中对DOM的操作都是操作的真实DOM但是这个真实DOM最终会被内存中的虚拟DOM覆盖导致不生效
  //   beforeMount() {
  //       console.log("beforeMount")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 常用的生命周期：Vue完成模板的解析并把初始的真实DOM元素放入页面后（挂载完毕）调用mounted，此时操作的真实DOM是生效的
  //   mounted() {
  //       console.log("mounted")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 数据发生更新时调用，但是此时数据是新的，但是页面上的数据还是是旧的
  //   beforeUpdate() {
  //       console.log("beforeUpdate")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 数据更新完成后调用，但是此时数据和页面上的数据都是新的
  //   updated() {
  //       console.log("updated")
  //       // 卡断点
  //       // debugger
  //   },

  //   // 常用的生命周期：当Vm调用vm.$destroy()后会调用beforeUnmount，此时vm的data、methods都是可用的，
  //   // 一般在此阶段关闭定时器、取消订阅消息、解绑自定义事件等收尾操作
  //   // 在beforeUnmount调用vm的data、methods修改数据是生效的，
  //   // 但是由于已经进入销毁流程了，之后不会调用beforeUpdate、updated、mounted所以数据不会同步到页面了
  //   beforeUnmount() {
  //       console.log("beforeUnmount")
  //       // 卡断点
  //       // debugger
  //   },

  //   // vm销毁完毕后调用unmounted
  //   unmounted() {
  //       console.log("unmounted")
  //       // 卡断点
  //       // debugger
  //   },
};
</script>
~~~


