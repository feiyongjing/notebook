# vue3的setup（setup会包含数据绑定、事件绑定、props、组件自定义事件绑定和触发）

### src/App.vue 
~~~vue
<template>
<Demo @hello="showHelloMsg" msg="你好啊" school="育才小学">

  <!-- 在vue3中具名插槽只能使用v-slot，而不能再写 slot="qwe" 了 -->
  <template v-slot:qwe>
    <span >前端工程师</span>
  </template>
</Demo>
</template>

<script>

import Demo from "./components/Demo.vue";

export default {
  name: "App",
  components: {Demo},

  setup(){
    function showHelloMsg(value){
      alert(`你好啊，你触发了hello事件，我收到的参数是：${value}！`)
    }

    return {
      showHelloMsg
    }
  }
 
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

### src/components/Demo.vue 
~~~vue
<template>
  <!-- vue3组中的模板可以没有根标签 -->
  <h1>一个人的信息</h1>
  <h2>姓名：{{ person.name }}</h2>
  <h2>年龄：{{ person.age }}</h2>
  <button @click="test">测试一下触发App组件的hello事件</button>
  <slot name="qwe"></slot>
</template>

<script>
import { ref, reactive } from "vue";

export default {
  name: "Demo",
  components: {},

  props: ["msg", "school"],
  emits: ["hello"],
  // vue2中的data、methods、等都写在setup中
  // setup一般情况下不能被async修饰，因为返回值不再是return的对象，而是promise对象，模板看不到return对象中的属性了
  // setup使用async并返回Promise对象的前提是该组件被异步引入并且使用Suspense标签包裹该组件标签
  // setup会在beforeCreate(最早的生命周期)之前执行一次，这时的this是undefined
  // setup的第一参数props是父组件传递的props，props必须和vue2一样在外面声明接收否则会有警告
  // setup的第二参数context中存储了一下上下文信息，其中主要是context.attrs、context.emit、context.slots
  // context.attrs是用于props的兜底，即如果没有向vue2一样在外面声明接收的属性数据会存放到这里面
  // context.emit是用于触发组件绑定的事件，但是必须在被绑定的组件中使用emits声明该组件绑定了哪些组件否则会出现警告
  // context.slots是用于存放父组件传递的全部插槽内容信息
  setup(props, context) {
    console.log("父组件传递的全部插槽内容信息", context.slots);

    // 直接定义变量数据
    let person = reactive({
      name: "张三",
      age: "18",
    });

    function test() {
      context.emit("hello", 666);
    }

    // 返回一个对象包含的data和函数可以直接在模板中使用
    // 由于属性名和属性变量名称一致所以 'person': person 可以简写为 person，同理test一样
    return {
      person,
      test,
    };
  },
};
</script>
~~~










