# vue常见UI组件库使用
- 移动端常见的Vue UI组件库
  - Vant使用参考官网：https://vant-ui.github.io/vant/v2/#/zh-CN/
  - Cube使用参考官网：https://didi.github.io/cube-ui/#/zh-CN 
  - Mint使用参考官网：https://mint-ui.github.io/#!/zh-cn
- PC端常见的Vue UI组件库
  - Element-UI使用参考官网：https://element.eleme.cn/#/zh-CN 
  - Iview-UI使用参考官网：https://www.iviewui.com/

## Element-UI使用，参考官网：https://element.eleme.cn/#/zh-CN/component/quickstart

### 安装element-ui
~~~shell
# 安装指定版本的element-ui
npm i element-ui@2.15.14

# 也可以直接在项目的package.json配置文件中写依赖库的版本，然后 npm i 安装所有项目依赖
{
  "name": "xxx",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    ...
  },
  "dependencies": {
    "element-ui": "^2.15.14",
    "vue": "^2.6.14",
  },
  ...
}
~~~

### 借助 babel-plugin-component 插件，可以只引入需要的Element-UI组件，以达到减小项目体积的目的
~~~shell
# 安装 babel-plugin-component
npm install babel-plugin-component -D

# 修改和package.json同级目录的babel.config.js配置文件，配置如下
# module.exports = {
#   presets: [
#     '@vue/cli-plugin-babel/preset',
#     ["@babel/preset-env", { "modules": false }]
#   ],
#   "plugins": [
#     [
#       "component",
#       {
#         "libraryName": "element-ui",
#         "styleLibraryName": "theme-chalk"
#       }
#     ]
#   ]
# }
~~~

### src/main.js 应用element-ui插件
~~~js
import Vue from 'vue'
import App from './App'

// 需要提前npm i element-ui 安装element-ui
// 引入ElementUI组件库和全部样式，缺点是实际开发根本用不到全部的组件和样式，会导致用户浏览器下载展示的js文件过大
// import ElementUI from 'element-ui';
// import 'element-ui/lib/theme-chalk/index.css';

// 按照需要引入对应的ElementUI组件
import { Button, Row,DatePicker } from 'element-ui';


// 关闭Vue的生产提示
Vue.config.productionTip = false

// 应用ElementUI全部插件
// Vue.use(ElementUI);

// 按需引用ElementUI组件
Vue.use(Button)
Vue.use(Row)
Vue.use(DatePicker)

const vm=new Vue({
  render: h => h(App),
}).$mount('#app')
~~~

### src/App.vue App组件管理其他组件，例如案例中的Banner组件
~~~vue
<template>
  <div>
    <Banner/>
  </div>
</template>

<script>
import Banner from './components/Banner.vue';

export default {
  name: "App",
  components: {
    Banner,
  },
};
</script>
~~~

### src/components/Banner.vue Banner组件使用element-ui
~~~vue
<template>
  <div>
    <button>原生的按钮</button>
    <input type="text" />

    <el-row>
      <el-button>默认按钮</el-button>
      <el-button type="primary">主要按钮</el-button>
      <el-button type="success">成功按钮</el-button>
      <el-button type="info">信息按钮</el-button>
      <el-button type="warning">警告按钮</el-button>
      <el-button type="danger">危险按钮</el-button>
    </el-row>

    <el-row>
      <el-button plain>朴素按钮</el-button>
      <el-button type="primary" plain>主要按钮</el-button>
      <el-button type="success" plain>成功按钮</el-button>
      <el-button type="info" plain>信息按钮</el-button>
      <el-button type="warning" plain>警告按钮</el-button>
      <el-button type="danger" plain>危险按钮</el-button>
    </el-row>

    <el-row>
      <el-button round>圆角按钮</el-button>
      <el-button type="primary" round>主要按钮</el-button>
      <el-button type="success" round>成功按钮</el-button>
      <el-button type="info" round>信息按钮</el-button>
      <el-button type="warning" round>警告按钮</el-button>
      <el-button type="danger" round>危险按钮</el-button>
    </el-row>

    <el-row>
      <el-button icon="el-icon-search" circle></el-button>
      <el-button type="primary" icon="el-icon-edit" circle></el-button>
      <el-button type="success" icon="el-icon-check" circle></el-button>
      <el-button type="info" icon="el-icon-message" circle></el-button>
      <el-button type="warning" icon="el-icon-star-off" circle></el-button>
      <el-button type="danger" icon="el-icon-delete" circle></el-button>
    </el-row>
    <div class="block">
      <span class="demonstration">带快捷选项</span>
      <el-date-picker
        v-model="value2"
        type="datetimerange"
        :picker-options="pickerOptions"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        align="right"
      >
      </el-date-picker>
    </div>
  </div>
</template>

<script>
export default {
  name: "Banner",
  data() {
    return {
      pickerOptions: {
        shortcuts: [
          {
            text: "最近一周",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            },
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            },
          },
        ],
      },
      value1: [new Date(2000, 10, 10, 10, 10), new Date(2000, 10, 11, 10, 10)],
      value2: "",
    };
  },
};
</script>
~~~






