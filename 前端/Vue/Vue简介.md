# Vue是什么？
Vue是一套用于构建用户界面的渐进式JavaScript框架，渐进式的意思是复杂的页面可以由多个组件来拼接组成

## Vue的特点
- 组件式开发，代码复用率高
- 声明式编码，无效手动操作DOM，提供代码开发效率
- 虚拟DOM + 优秀的Diff算法，当数据发生变化之后根据Diff算法比较新旧数据，在尽量复用Dom节点的情况下变动数据发生变化的Dom节点

### vue官网：https://cn.vuejs.org/

### 推荐Vue Devtools浏览器插件安装，这样方便在浏览器查看和调试 Vue 应用

# Vue安装方式
- 方式一：直接使用script标签引入vue的代码或者提前下载下来放入自定义文件引入
- 方式二：使用vue-cli创建vue项目
- 方式三：使用vue3的vite创建vue项目

### 方式一：直接使用script标签引入vue的代码
~~~html
<!-- 开发版本 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.16/dist/vue.js"></script>

<!-- 生成环境版本，相比开发版本进行了压缩，删除了警告，不便于调试 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.7.16"></script>
~~~








