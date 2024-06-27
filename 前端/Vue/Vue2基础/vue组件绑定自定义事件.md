# vue组件绑定自定义事件

## 组件绑定自定义事件使用
### src/App.vue 设置给子组件绑定自定义事件
~~~vue
<template>
  <div id="app">
    <h1>你好啊</h1>
    <!-- 通过父组件给子组件传递函数类型的props实现子组件给父组件传递数据 -->
    <School :getSchoolName="getSchoolName" />

    <!-- 以下是两种子组件绑定自定义事件给父组件传递数据 -->
    <!-- 通过父组件给子组件直接绑定一个自定义事件实现子组件给父组件传递数据 -->
    <Student @getStudentName="getStudentName" />
    <!-- 通过父组件在完成挂载后给子组件绑定一个自定义事件实现子组件给父组件传递数据 -->
    <Teacher ref="teacher"/>
  </div>
</template>

<script>
import School from "./components/School.vue";
import Student from "./components/Student.vue";
import Teacher from "./components/Teacher.vue";

export default {
  name: "App",
  components: {
    School,
    Student,
    Teacher,
  },
  data() {
    return {};
  },

  methods: {
    getSchoolName(schoolName) {
      alert("父组件收到SchoolName:" + schoolName);
    },

    getStudentName(studentName) {
      alert("父组件收到StudentName:" + studentName);
    },
    getTeacherName(teacherName) {
      alert("父组件收到TeacherName:" + teacherName);
    },
  },


  mounted(){
    // 通过Teacher组件vc对象的$on方法绑定一个自定义事件，第一参数是事件名称，后面的参数事件的回调函数
    this.$refs.teacher.$on("getTeacherName",this.getTeacherName)
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background-color: gray;
  text-align: center;
}
</style>
~~~

### src/components/School.vue 通过父组件给子组件传递函数类型的props实现子组件给父组件传递数据
~~~vue
<template>
  <div class="school">
    <div>学校名称：{{ name }}</div>
    <div>学校地址：{{ address }}</div>
    <button @click="sendSchoolName()">把学校名称给APP组件</button>
  </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      name: "育才小学",
      address: "xxx省xxx市xxx区xxx街道23号",
    };
  },
  methods: {
    sendSchoolName() {
      this.getSchoolName(this.name);
    },
  },

  props: ["getSchoolName"],
};
</script>

<style scoped>
.school {
  background-color: aquamarine;
}
</style>
~~~

### src/components/Student.vue 通过父组件给子组件直接绑定一个自定义事件实现子组件给父组件传递数据
~~~vue
<template>
  <div class="student">
    <div>学生名称：{{ name }}</div>
    <div>学生性别：{{ sex }}</div>
    <div>学生年龄：{{ myAge }}</div>
    <button @click="sendStudentName()">把学生名称给APP组件</button>
  </div>
</template>

<script>
export default {
  name: "Student",

  data() {
    return {
      name: "张三",
      sex: "男",
      myAge: 18,
    };
  },

  methods: {
    sendStudentName() {
      // $emit触发Student组件实例身上的getStudentName事件并且传递触发事件的参数
      this.$emit("getStudentName", this.name);
    },
  },
};
</script>

<style scoped>
.student {
  background-color: blueviolet;
}
</style>
~~~

### src/components/Teacher.vue 通过父组件在完成挂载后给子组件绑定一个自定义事件实现子组件给父组件传递数据
~~~vue
<template>
  <div class="teacher">
    <div>教师名称：{{ name }}</div>
    <div>教师性别：{{ sex }}</div>
    <div>教师年龄：{{ myAge }}</div>
    <button @click="sendTeacherName()">把教师名称给APP组件</button>
  </div>
</template>

<script>
export default {
  name: "Teacher",

  data() {
    return {
      name: "李四",
      sex: "男",
      myAge: 18,
    };
  },

  methods: {
    sendTeacherName() {
      // $emit触发Teacher组件实例身上的getTeacherName事件并且传递触发事件的参数
      this.$emit("getTeacherName", this.name);
    },
  },
};
</script>

<style scoped>
.teacher {
  background-color: rgb(166, 251, 149);
}
</style>
~~~

## 在Vue原型上全局设置自定义事件，可以使所有组件使用事件（一般很少使用）
### 安装全局数据总线
~~~js
import Vue from 'vue'
import App from './App.vue'
// 关闭Vue的生产提示
Vue.config.productionTip = false

new Vue({
  render: h => h(App),

  beforeCreate(){
    // 安装全局数据总线，所有组件可以拿到Vue原型上的$bus，$bus应用当前应用的Vm后所有组件就可以操作$bus绑定事件和触发、解绑事件
    Vue.prototype.$bus=this  
  }
}).$mount('#app')

~~~

### src/App.vue 
~~~vue
<template>
  <div id="app">
    <h1>你好啊</h1>
    <School  />
    <Student />
    <Teacher />
  </div>
</template>

<script>
import School from "./components/School.vue";
import Student from "./components/Student.vue";
import Teacher from "./components/Teacher.vue";

export default {
  name: "App",
  components: {
    School,
    Student,
    Teacher,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background-color: gray;
  text-align: center;
}
</style>
~~~

### src/components/School.vue 绑定自定义事件到全局事件总线
~~~vue
<template>
  <div class="school">
    <div>学校名称：{{ name }}</div>
    <div>学校地址：{{ address }}</div>
  </div>
</template>

<script>
export default {
  name: "School",
  data() {
    return {
      name: "育才小学",
      address: "xxx省xxx市xxx区xxx街道23号",
    };
  },
  methods: {
    sendTeacherName(teacherNam) {
      alert("学校组件接收到了教师组件发送的教师名称：" + teacherNam);
    },
  },

  mounted() {
    // 当前组件绑定事件到全局事件总线，第一参数是事件名称，后面的参数事件的回调函数
    this.$bus.$on("sendTeacherName", this.sendTeacherName);
  },

  beforeDestroy() {
    // 由于vc的销毁不会自动解绑全局事件中心的自定义事件，所以需要手动解绑，避免vc销毁了但是全局事件总线还残留了与它绑定的自定义事件
    this.$bus.$off("sendTeacherName");
  },
};
</script>

<style scoped>
.school {
  background-color: aquamarine;
}
</style>
~~~

### src/components/Student.vue 触发全局事件总线的自定义事件
~~~vue
<template>
  <div class="student">
    <div>学生名称：{{ name }}</div>
    <div>学生性别：{{ sex }}</div>
    <div>学生年龄：{{ myAge }}</div>
    <button @click="sendStudentName()">把学生名称给Teacher教师组件</button>
  </div>
</template>

<script>
export default {
  name: "Student",

  data() {
    return {
      name: "张三",
      sex: "男",
      myAge: 18,
    };
  },

  methods: {
    sendStudentName() {
      // 触发全局事件总线的事件，第一参数是事件名称，后面的参数是触发事件时携带的参数
      this.$bus.$emit("sendStudentName",this.name)
    },
  },
};
</script>

<style scoped>
.student {
  background-color: blueviolet;
}
</style>
~~~

### src/components/Teacher.vue 绑定自定义事件到全局事件总线和触发全局事件总线的自定义事件
~~~vue
<template>
  <div class="teacher">
    <div>教师名称：{{ name }}</div>
    <div>教师性别：{{ sex }}</div>
    <div>教师年龄：{{ myAge }}</div>
    <button @click="sendTeacherName()">把教师名称给School组件</button>
  </div>
</template>

<script>
export default {
  name: "Teacher",

  data() {
    return {
      name: "李四",
      sex: "男",
      myAge: 18,
    };
  },

  methods: {
    sendTeacherName() {
      // 触发全局事件总线的事件，第一参数是事件名称，后面的参数是触发事件时携带的参数
      this.$bus.$emit("sendTeacherName", this.name);
    },

    sendStudentName(studentName) {
      alert("教师组件接收到了学生组件发送的学生名称：" + studentName);
    },
  },
  mounted() {
    // 当前组件绑定事件到全局事件总线，第一参数是事件名称，后面的参数事件的回调函数
    this.$bus.$on("sendStudentName", this.sendStudentName);
  },
  beforeDestroy() {
    // 由于vc的销毁不会自动解绑全局事件中心的自定义事件，所以需要手动解绑，避免vc销毁了但是全局事件总线还残留了与它绑定的自定义事件
    this.$bus.$off("sendStudentName");
  },
};
</script>

<style scoped>
.teacher {
  background-color: rgb(166, 251, 149);
}
</style>
~~~





