### 安装vuex插件
~~~shell
npn i vuex
~~~

## 使用vuex插件传输数据

### 项目目录如下
~~~shell
/
└── src
    ├── components
    │   ├── Count.vue
    │   └── Person.vue
    ├── store
    │   ├── index.js
    │   ├── count.js    
    │   └── person.js
    ├── App.vue
    └── main.js
~~~

### src/main.js 应用Vuex的store配置
~~~js
import Vue from 'vue'
import App from './App.vue'

// 使用vuex需要引入自己编写的store，如果是在index.js文件就只需要提供目录即可，默认会在目录下查找index.js文件
import store from './store/index.js'

// 关闭Vue的生产提示
Vue.config.productionTip = false

const vm=new Vue({
  // store:store 使用自己编写的store，简写为store
  store,
  render: h => h(App),
}).$mount('#app')
~~~

### src/store/index.js 该文件用于引入Vuex插件应用和创建vuex中最为核心的store模块
~~~js
import Vue from 'vue'
// 引入自定义的store模块
import CountOptions from './count'
import personOptions from './person'

// 引入Vuex
import Vuex from 'vuex'
// vuex插件必须先应用然后再创建store
Vue.use(Vuex)

// 创建store并暴露store
export default new Vuex.Store({
    // 添加不同模块的的store，属性名是自定义的模块名称，属性值是store模块对象
    // 注意模块对象上必须有namespaced:true 属性设置
    modules:{
        countAbout:CountOptions,
        personAbout:personOptions,
    }
})
~~~

### src/store/count.js 自定义的store模块
~~~js
// store中包含actions用于复杂操作state中的数据
// mutations用于简单操作数据
// state用于存储数据
// getters用于加工计算属性
// 修改或是查看vuex state数据链路是先 vm.$store.dispatch 选择调用 actions中的函数
// 然后actions中的函数通过默认的第一参数context, context.commit选择调用 mutations中的函数
// 最后在mutations中的函数通过默认的第一参数state，修改state中的数据
// 如果是简单的修改或是查看可以省略调用actions中的函数，而是直接this.$store.commit 选择调用mutations中的函数
export default {

    // 开启模块化，当前store是一个独立的模块
    namespaced:true,

    // 准备actions，用于复杂操作state中的数据，复杂的业务可以写在actions的函数中
    actions: {
        // 可以从第一参数context中获取一些信息，比如state数据和commit函数
        jiaOdd(context, value) {
            console.log("actions的context参数上下文",context)
            if (context.state.sum % 2) {
                // 通过commit选择调用mutations函数和传递参数
                context.commit("jia", value)
            }

        },

        jiaWait(context, value) {
            setTimeout(() => {
                context.commit("jia", value)
            }, 500);
        }
    },

    // 准备mutations，用于简单操作state中的数据，简单的业务修改写在mutations中
    mutations: {
        // 第一个参数默认是state数据，后面的参数是commit传入的
        jia(state, value) {
            state.sum += value
        },
       
        jian(state, value) {
            state.sum -= value
        },
    },

    // 准备state，用于存储数据
    state: {
        sum: 0,
        school: '育才初级中学',
        subject: "物理",
    },

    // 加工计算属性，可以方便的计算
    getters: {
        // 通过vm.$store.getters.bigsum就可以调用该计算属性了
        bigSum(state) {
            return state.sum * 10
        }
    }
}
~~~

### src/store/person.js 自定义的store模块
~~~js
import axios from 'axios'

export default {
    namespaced: true,
    actions: {
        addPersonServer(context) {
            // API用于生成随机的名称，可能不可用
            axios.get('https://xxx.com/xxx').then(
                response => {
                    context.commit('addPerson', { id: Date.now(), name: response.data })
                }
                ,
                error => {
                    alert(error.message)
                }
            )
        }
    },
    mutations: {
        addPerson(state, value) {
            state.personList.unshift(value)
        }
    },
    state: {
        personList: [
            {
                id: Date.now(),
                name: "张三"
            }
        ],
    },
    getters: {}
}
~~~

### src/App.vue App组件管理其他组件
~~~vue
<template>
  <div id="app">
    <Count />
    <hr />
    <Person />
  </div>
</template>

<script>
import Count from "./components/Count.vue";
import Person from "./components/Person.vue";

export default {
  name: "App",
  components: {
    Count,
    Person,
  },
  mounted() {
    console.log(this);
  },
};
</script>
~~~

### src/components/Count.vue Count组件获取和修改自定义的store模块中的数据
~~~vue
<template>
  <div class="count">
    <h1>当前求和为{{ sum }}</h1>
    <h1>当前求和放大10倍为{{ bigSum }}</h1>
    <h1>我在{{ school }}学习{{ subject }}</h1>
    <h1>Person组件的人员数量是{{ personList.length }}</h1>
    <select v-model="n">
      <option :value="1">1</option>
      <option :value="2">2</option>
      <option :value="3">3</option>
    </select>
    <button @click="increment(n)">+</button>
    <button @click="decrement(n)">-</button>
    <button @click="incrementOdd(n)">当前求和为奇数再+</button>
    <button @click="incrementWait(n)">等待一段时间后再+</button>
  </div>
</template>

<script>
// 引入mapState使用简写形式从vuex中获取state的数据并生成对应的代码调用该函数
// 引入mapGetters使用简写形式从vuex中获取state的数据Getters计算属性并生成对应的代码调用该函数
// 引入mapMutations使用简写形式从vuex中获取Mutations的函数并生成对应的代码调用该函数
// 引入mapActions使用简写形式从vuex中获取Actions的函数并生成对应的代码调用该函数
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";

export default {
  name: "Count",
  data() {
    return {
      n: 1,
    };
  },


  computed: {
    // 从vuex的store中获取state的属性数据
    // 方式一：手动一个个编写获取store模块中的state数据
    // 注意如果使用的是模块化的store，this.$store.state.模块名称.state属性名称 获取模块化store下的state的属性数据
    // sum(){
    //   return this.$store.state.countAbout.sum
    // },
    // school(){
    //   return this.$store.state.countAbout.school
    // },
    // subject(){
    //   return this.$store.state.countAbout.subject
    // },
    // personList(){
    //   return this.$store.state.personAbout.personList
    // },

    // 方式二：引入mapState使用简写形式从vuex中获取state的数据并生成对应的代码调用该函数
    // 注意如果使用的是模块化的store，mapState的第一个参数必须指定store的模块名称
    // 数组写法，只有当computed中的函数名称和state中的属性一致时可以这样写
    ...mapState("countAbout", ["sum", "school", "subject"]),
    ...mapState("personAbout", ["personList"]),
    // 对象写法，对象的属性名是生成computed中的函数名称，属性值是state中的属性名称并且必须是字符串
    // ...mapState('countAbout',{sum:"sum", school:"school", subject:"subject"}),



    // 从vuex的store中获取state的Getters计算属性
    // 方式一：手动一个个编写获取store模块中的state数据
    // 注意如果使用的是模块化的store，通过this.$store.getters['模块名称/计算属性名称']获取store的计算属性
    // bigSum() {
    //   return this.$store.getters["countAbout/bigSum"];
    // },

    // 方式二：引入mapGetters使用简写形式从vuex中获取state的数据Getters计算属性并生成对应的代码调用该函数
    // 如果使用的是模块化的store，mapGetters的第一个参数必须指定store的模块名称
    // 数组写法，只有当omputed中的函数名称与state中的Getters计算属性名称一致时可以这样写
    ...mapGetters('countAbout', ["bigSum"]),
    // 对象写法，对象的属性名是生成computed中的函数名称，属性值是state中的Getters计算属性名称并且必须是字符串
    // ...mapGetters('countAbout',{bigSum:"bigSum"}),
  },

  methods: {
    // 调用vuex的store模块mutations中函数简单修改state数据
    // 方式一：手动一个个编写调用store模块mutations中函数修改state数据
    // 注意如果使用的是模块化的store，通过this.$store.commit第一函数 ['store模块名称/store模块mutations中函数名称'] 调用mutations中函数
    // increment(value) {
    //   // 调用$store的commit传递调用的函数名和参数会在store的mutations上查找对应的函数调用
    //   // commit只能调用mutations中函数处理简单的直接处理store上的state数据
    //   this.$store.commit("countAbout/jia", value);
    // },
    // decrement(value) {
    //   this.$store.commit("countAbout/jian", value);
    // },

    // 方式二：引入mapMutations使用简写形式从vuex中获取Mutations的函数并生成对应的代码调用该函数
    // 如果使用的是模块化的store，mapMutations的第一个参数必须指定store的模块名称，并且生成的methods中的函数参数和Mutations的函数参数一致
    // 对象写法，对象的属性名是生成methods中的函数名称，属性值是state中的Mutations的函数名称并且必须是字符串
    ...mapMutations('countAbout',{increment:"jia",decrement:"jian"}),
    // 数组写法，只有当生成methods中的函数名称与state中的Mutations的函数名称一致时可以使用
    // 注意目前methods中的函数名称与state中的Mutations的函数名称不一致，所以没有使用数组写法
    // ...mapMutations('countAbout',["increment","decrement"]),


    
    // 调用vuex的store模块actions中函数复杂修改state数据
    // 方式一：手动一个个编写调用store模块actions中函数修改state数据
    // 注意如果使用的是模块化的store，通过this.$store.dispatch第一函数 ['模块名称/dispatch中函数名称'] 调用dispatch中函数
    // incrementOdd(value) {
    //   // 调用$store的dispatch传递调用的函数名和参数会在store的actions上查找对应的函数调用
    //   // store的actions中的函数用于处理一些复杂的业务，actions中的函数内部可能会调用commit在store的mutations上查找对应的函数调用
    //   // 最终修改store上的state数据
    //   this.$store.dispatch("countAbout/jiaOdd", value);
    // },
    // incrementWait(value) {
    //   this.$store.dispatch("countAbout/jiaWait", value);
    // },

    // 引入mapActions使用简写形式从vuex中获取Actions的函数并生成对应的代码调用该函数
    // 如果使用的是模块化的store，mapActions的第一个参数必须指定store的模块名称，并且生成的methods中的函数参数和Actions的函数参数一致
    // 对象写法，对象的属性名是生成methods中的函数名称，属性值是state中的Actions的函数名称并且必须是字符串
    ...mapActions('countAbout',{incrementOdd:"jiaOdd",incrementWait:"jiaWait"}),
    // 数组写法，只有当生成methods中的函数名称与state中的Actions的函数名称一致时可以使用
    // 注意目前methods中的函数名称与state中的Actions的函数名称不一致，所以没有使用数组写法
    // ...mapActions('countAbout',["incrementOdd","incrementWait"]),
  },
};
</script>

<style scoped>
button {
  margin: 5px;
}
</style>
~~~

### src/components/Person.vue Person组件获取和修改自定义的store模块中的数据
~~~vue
<template>
<div>
    <h1>人员列表</h1>
    <h1>Count组件的求和是{{sum}}</h1>
    <input type="text" placeholder="请输入名字" v-model="name">
    <button @click="add">添加用户</button>
    <button @click="addPersonServer">添加一个用户，用户名称随机</button>
    <ul>
        <li v-for="p in personList" :key="p.id">{{p.name}}</li>
    </ul>
</div>
</template>

<script>
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";

export default {
  name: "Person",
  data() {
    return {
      name:""
    };
  },
  computed: {
    ...mapState('countAbout',["sum"]),
    ...mapState('personAbout',["personList"]),
  },
  methods:{
    add(){
        const personObj={id:Date.now(),name:this.name}
        // 注意如果使用的是模块化的store，第一参数通过 模块名称/函数名称 调用对应store模块化Mutations中的函数
        this.$store.commit("personAbout/addPerson",personObj)
        this.name=""
    },
    ...mapActions('personAbout',["addPersonServer"])
  }
};
</script>
~~~










