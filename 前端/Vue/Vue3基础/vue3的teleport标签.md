# vue3的teleport标签（瞬间移动标签到其他标签内部）

### src/components/Dialog.vue 
~~~vue
<template>
  <div>
    <button @click="isShow = true">点我弹个窗</button>

    <!-- teleport会将内容中的标签全部移动到to所指定的位置，例如当前是移动到body标签中了 -->
    <teleport to="body">
      <div class="mask" v-show="isShow">
        <div class="dialog">
          <h3>我是一个弹窗</h3>
          <h4>一些内容</h4>
          <h4>一些内容</h4>
          <h4>一些内容</h4>
          <button @click="isShow = false">关闭弹窗</button>
        </div>
      </div>
    </teleport>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  name: "Dialog",
  setup() {
    let isShow = ref(false);

    return { isShow };
  },
};
</script>

<style scoped>
.mask {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.5);
}
.dialog {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;

  width: 300px;
  height: 300px;
  background-color: green;
}
</style>
~~~



