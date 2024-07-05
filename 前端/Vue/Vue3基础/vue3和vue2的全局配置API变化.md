# vue3和vue2的全局配置API变化（例如插件使用）
- 在vue3中是通过App实例对象进行的全局配置

### 插件变化
~~~js
import { createApp } from 'vue'
import App from './App.vue'

import {myPlugin} from './js/myPlugin.js'  // 引用自定义插件

// 创建一个App实例对象（类似于Vue2的vm组件实例对象，但是比vm要更'轻'，即对象上的属性和方法要少一些）
const app=createApp(App)

// 应用插件，在vue2中是操作的Vue实例对象，但是vue3中是操作的App实例对象
// 其他的全局配置也都同理类似，在vue3中是通过App实例对象进行的全局配置
app.use(myPlugin)

// App实例对象挂载到对应的Dom节点
app.mount('#app')
~~~


