# Vue组件使用
- 非单文件组件使用：在一个文件编写多个vue组件，缺点无法复用、html模板编写没有高亮、代码一多就观看麻烦
- 单文件组件使用：在一个vue文件编写一个vue组件，相比非单文件组件的好处是html、css、js，在文件中分离并且一个文件一个组件方便复用组件

## 非单文件组件使用例子
~~~html
<div id="root">
    <!-- 非单文件组件例子 -->
    <div></div>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    // 使用组件第一步：创建student组件
    const student = Vue.extend({
        // el: ".xxx", 组件定义时，一定不要写el配置项，因为最终所有的组件都要被一个Vm管理，由vm决定服务于谁
        // name:'组件的name属性设置浏览器插件vue开发者工具中显示的组件名称，但是无法影响组件注册和在html中使用时的组件名称',
        // 组件对应的html模板，注意根节点只能有一个，如果有多个请在外面包裹一层div
        template: `
            <div>
                <h3>学生名称：{{studentName}}</h3>
                <h3>学生年龄：{{age}}</h3>
            </div>
        `,

        // 在组件的定义中data需要写成函数，函数的返回值是最终需要的数据
        // 原因是组件有多处地方使用导致需要多份data数据并且互不干扰
        data() {
            return {
                studentName: "张三",
                age: 18,
            }
        },
    })

    // 创建school组件并嵌套student组件，创建组件的简写形式，但是组件在注册时还是会自动调用Vue.extend
    // 嵌套组件第一步：注意student子组件需要提前创建好
    const school = {
        // 嵌套组件第三步：注意嵌套student组件后使用子组件需要在模板中使用子组件标签
        template: `
            <div>
               <h3>学校名称：{{schoolName}}</h3>
               <h3>学校地址：{{address}}</h3>
               <button @click="showName">点我弹出学习名称</button>
               <!-- 直接将子组件的名称当成标签使用 -->
               <student><student>
            </div>
        `,

        data() {
            return {
                schoolName: "育才小学",
                address: "xxx省xxx市xxx区xxx街道23号",
            }
        },

        methods: {
            showName() {
                alert(this.schoolName)
            }
        },

        // 嵌套组件第二步：注册子组件
        components: {
            student
        }
    }

    // 使用第二步：注册全局组件，在所有的Vm前注册就可以在所有的Vm中都能使用
    // Vue.component("school",school)
    let vm = new Vue({
        el: "#root",
        // 使用第二步：注册组件（局部注册）。第三步：直接将组件的名称当成标签在html中使用
        // 如果key是一个单词那么可以写成全小写或首字母大写的，第三步按照key使用即可
        // 如果是多个单词组成的key,
        // 第一种写法是'my-school' 第三步使用<my-school></my-school> 但是注意在浏览器的Vue开发者插件中显示的组件名称是MySchool
        // 第二种写法是'MySchool' 第三步使用<MySchool></MySchool> 但是必须是在脚手架环境下使用
        // 浏览器控制台vm.$children会看到vm下的组件，而组件下的$children会看到向下嵌套的组件
        components: {
            school,  // school:school  key是最终组件的名称，value是组件变量，如果它们的名称一致就可以简写为key
            student
        },

        // 组件使用第三步：如果不行在html中使用组件标签那么请在vm中使用template模板添加组件
        //  注意可以组件使用可以写成以下的自闭合单标签，但是注意必须是在脚手架环境下使用，否则会导致后面第二个组件无法渲染
        //     <school/>
        //     <school/> 
        template: `
            <div>
            <school></school>
            </div>
        `,
    })

    // school和student组件本质都是一个名为VueComponent的构造函数，VueComponent不是程序员定义的，是调用Vue.extend生成的。
    // 每次调用Vue.extend返回的都是一个全新的VueComponent
    // 当html或者是模板的<school/>或者是<school></school>被Vue解析时会创建school组件的实例对象，
    // 即Vue执行 new VueComponent(options)生成VueComponent实例对象(简称vc)
    // 在组件配置中：data函数、methods中的函数、watch中的函数、computed中的函数，它们的this都是VueComponent实例对象，即vc
    // 在new Vue(options)配置中：data属性、methods中的函数、watch中的函数、computed中的函数，它们的this都是Vue实例对象，即vm
    // 注意VueComponent.prototype.__proto__ === Vue.prototype 即vc的原型对象上的隐式原型属性就是vm的原型对象
    // 也就是vc可以访问到Vue原型上的属性和方法
</script>
~~~

## 单文件组件使用例子如下，但是如下代码必须在vue-cli框架中才能正常运行
### index.html主页面
~~~html
<div id="root">
    <!-- 单文件组件例子 -->
    <App></App>
</div>

<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="model" src="../js/main.js"></script> <!-- main.js创建Vue实例对象并且注册App组件 -->
~~~

### main.js创建Vue实例对象并且注册App组件
~~~js
import App from "../vue/App.vue";  // App注册组件代码

Vue.config.productionTip = false

new Vue({
    el: "#root",
    
    components: {
        App,  // App:App  key是最终组件的名称，value是组件变量，如果它们的名称一致就可以简写为key
    },
})
~~~

### App组件代码
~~~vue
<template>
  <div class="school">
    <h3>学校名称：{{ schoolName }}</h3>
    <h3>学校地址：{{ address }}</h3>
    <button @click="showName">点我弹出学习名称</button>
  </div>
</template>

<script>
  export default {
    name: "App",

    // 在组件的定义中data需要写成函数，函数的返回值是最终需要的数据
    // 原因是组件有多处地方使用导致需要多份data数据并且互不干扰
    data() {
      return {
        schoolName: "育才小学",
        address: "xxx省xxx市xxx区xxx街道23号",
      };
    },
    methods: {
      showName() {
        alert(this.schoolName);
      },
    },
  };
</script>

<style>
.school {
  width: 100px;
  height: 100px;
  background-color: aquamarine;
}
</style>
~~~














