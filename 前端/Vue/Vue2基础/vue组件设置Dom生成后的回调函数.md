# vue组件的$nextTick设置Dom生成后的回调函数

### src/App.vue App组件管理其他组件
~~~vue
<template>
  <div id="app">
    <Todo></Todo>
  </div>
</template>

<script>
import Todo from "./components/Todo.vue";

export default {
  name: "App",
  components: {
    Todo,
  },
};
</script>

<style>
#app {
  background-color: gray;
}
</style>

~~~

### src/components/Todo.vue Todo组件通过$nextTick设置Dom生成后的回调函数
~~~vue
<template>
  <div>
    <span v-if="!isEdit">{{ title }}</span>
    <input
      v-else
      type="text"
      ref="inputTitle"
      v-model="title"
      style="width:500px;"
      @blur="headleBlur()"
    />

    <button v-if="!isEdit" @click="headleEdit()">编辑</button>
    <button v-else @click="headleBlur()">提交</button>
  </div>
</template>

<script>
export default {
  name: "Todo",
  data() {
    return {
      title: "123",
      isEdit: false,
    };
  },

  methods: {
    // 编辑
    headleEdit() {
      this.isEdit = true;

      // 由于修改isEdit后没有马上就生效产生编辑input的DOM，这个时候无法获取焦点
      // 需要使用 $nextTick 进行指定在下一次Dom生成后的回调函数，在该函数中可以获取到编辑input的焦点
      this.$nextTick(function () {
        // 获取编辑input框的焦点
        this.$refs.inputTitle.focus();
      });
    },

    // 编辑框失去焦点时触发
    headleBlur() {
      this.isEdit = false;

      if (this.title.trim().length === 0) {
        alert("输入不能为空！");
      } else {
        alert("输入是" + this.title);
      }
    },
  },
};
</script>
~~~


