# props接收父组件传递的数据，例子如下
### 父组件
~~~vue
<template>
  <div>
    <div class="row">
      <!-- 通过子组件标签属性传递数据，如果是数字需要使用v-bind绑定，以防止数据被当成字符串 --> 
      <School studentName="张三" :age="18"></School>
    </div>
  </div>
</template>

<script>
import School from "./components/School.vue";
export default {
  name: "App",
  components: {
    School,
  },
};
</script>

<style scoped>
</style>
~~~

### 子组件
~~~vue
<template>
    <div class ="school">
        <h3>学校名称：育才小学</h3>
        <!-- 使用插值语法获取数据，如果data中没有就去props中取父元素传递的属性，
             但如果data的属性和props的属性发生冲突则最终数据以props的属性值为准 -->
        <h3>学生名称：{{ studentName }}</h3>
        <h3>学生年龄：{{ age+1 }}</h3>
    </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
    };
  },

  // props设置父组件传递的数据属性名称，
  // 注意不要直接修改父组件传递的数据否则会控制台报错出现奇怪的问题，如果需要修改请先保存数据到data属性中，然后直接修改data的属性
  // 方式一：简单接收父元素数据  
  // props: ["studentName", "age"],
  
  // 方式二：接收父元素数据和类型申明限制
  // props: {
  //   studentName: String, 
  //   age: Number
  // },

  // 方式三：接收父元素数据和类型申明限制，并且限制属性数据是否必传和默认值设置
  props: {
    studentName: {
        type: String,
        required: true
    }, 
    age: {
        type: Number,
        default: 13
    }
  },
};
</script>

<style>
.school {
  background-color: aquamarine;
}
</style>
~~~



