# vue的ref使用
~~~vue
<template>
    <div class ="school">
        <!-- 元素设置ref属性，方便从vm对象获取到元素 -->
        <h3 ref="schoolName">学校名称：育才小学</h3>
        <button @click="showName">点我弹出学校名称</button>
    </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
    };
  },

  methods: {
    showName() {
      // 无需使用Dom操作，直接通过vm对象的$ref属性获取模板中带有指定ref属性值的元素或者子组件vm对象
      alert(this.$refs.schoolName.innerHTML);
    },
  },
};
</script>

<style>
.school {
  background-color: aquamarine;
}
</style>
~~~

