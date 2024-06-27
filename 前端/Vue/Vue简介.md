# Vue是什么？
Vue是一套用于构建用户界面的渐进式JavaScript框架，渐进式的意思是复杂的页面可以由多个组件来拼接组成

## Vue的特点
- 组件式开发，代码复用率高
- 声明式编码，无效手动操作DOM，提供代码开发效率
- 虚拟DOM + 优秀的Diff算法，当数据发生变化之后根据Diff算法比较新旧数据，在尽量复用Dom节点的情况下变动数据发生变化的Dom节点

### vue官网：https://cn.vuejs.org/

## Vue的插件
- 推荐Vue Devtools浏览器插件安装，这样方便在浏览器查看和调试 Vue 应用
- Vscoe 安装Vue 3 Snippets插件
- 

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


# vue学习路线
- vue数据绑定
- vue事件绑定
- vue计算属性
- vue监视属性
- vue条件渲染
- vue列表渲染
- vue过滤器
- vue对象生命周期
- vue组件使用
- vue-cli框架使用
- vue通过ref查找元素或者vm对象
- props接收父组件传递的数据
- mixins混入复用公共代码
- vue插件使用
- vue组件模板的style样式设置
- vue组件绑定自定义事件实现子组件向父组件传递数据
- vue组件的$nextTick设置Dom生成后的回调函数
- animate动画库使用: https://animate.style
- vue配置后端服务的代理
- vue的插槽：默认插槽、具名插槽、作用域插槽
- vuex插件传输组件之间的数据
- vue路由组件基础使用
- vue多层嵌套路由组件使用
- 

# vue常见指令
- v-bind      单向绑定元素属性
- v-model     双向绑定表单元素属性
- v-on        绑定事件函数
- v-show      控制元素是否显示
- v-if        控制元素是否显示
- v-if-else   控制元素是否显示
- v-else      控制元素是否显示
- v-for       数据遍历展开为多个元素
- v-text      设置元素内容被data属性数据进行纯文本完全覆盖替换，建议使用{{}}插值语法进行部分替换
- v-html      设置元素内容被data属性数据进行替换（如果是html文本会被解析为元素），建议使用{{}}插值语法进行部分替换
- v-cloak     配合css使用隐藏带由v-cloak属性的元素，直到Vue的js库引入成功（网络差的情况会出现满屏的{{}}元素插值内容）后vue自动去除标签的v-cloak属性
- v-once      v-once指定的节点在初次动态的渲染后被认为是静态内容，不会再发生变化了
- v-pre       Vue跳过了带有v-pre属性的元素的编译，所以这些元素Vue无法影响
- ref         Vue的ref属性替代dom查找元素




