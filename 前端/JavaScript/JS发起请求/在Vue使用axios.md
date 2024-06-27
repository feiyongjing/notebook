~~~vue
<template>
  <div class="school">
    <button @click="getSchoolName()">获取学校名称</button>
    <button @click="getSchoolAddrees()">获取学校地址名称</button>
    <button @click="getSchoolCreateTime()">获取创办时间</button>
  </div>
</template>

<script>
// 引入axios
import axios from "axios";

export default {
  name: "School",
  data() {},
  methods: {
    getSchoolName() {
      axios({
        // 请求url路径，可以写http://xxx.com/url注意跨越问题，也可以省略域名只写路径/url但是会在当前的服务中查找这个路径的接口
        // 避免浏览器的跨域，在vue.config中开启代理，
        // 注意vue.config的代理配置，当url请求的资源前端没有的时候会转发给对应的代理配置
        // 目前/xxx前缀会代理到http://127.0.0.1:5000
        url: "/xxx/url",
        // 设置请求的类型
        method: "POST",
        // 跟在url后面的参数设置
        params: {
          id: 222,
        },
        // 设置请求头
        headers: {
          "Content-Type": "application/json; charset=UTF-8",
          a: 22,
        },
        // 设置请求body
        data: {
          username: "张三",
          password: "22",
        },
      }).then(
        // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
        (response) => {
          console.log("请求响应状态码", response.status);
          console.log("请求响应状态字符串", response.statusText);
          console.log("请求响应全部header头信息", response.headers);
          console.log("请求响应体信息", response.data);
        },
        (error) => {
          console.log("请求失败了", error.masage);
        }
      );
    },

    getSchoolAddrees() {
      axios({
        // 请求url路径，可以写http://xxx.com/url注意跨越问题，也可以省略域名只写路径/url但是会在当前的服务中查找这个路径的接口
        // 避免浏览器的跨域，在vue.config中开启代理，
        // 注意vue.config的代理配置，当url请求的资源前端没有的时候会转发给对应的代理配置
        // 目前/api/V1前缀会代理到http://127.0.0.1:5001
        url: "/api/V1/url",
        // 设置请求的类型
        method: "POST",
        // 跟在url后面的参数设置
        params: {
          id: 333,
        },
        // 设置请求头
        headers: {
          "Content-Type": "application/json; charset=UTF-8",
          a: 33,
        },
        // 设置请求body
        data: {
          username: "李四",
          password: "33",
        },
      }).then(
        // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
        (response) => {
          console.log("请求响应状态码", response.status);
          console.log("请求响应状态字符串", response.statusText);
          console.log("请求响应全部header头信息", response.headers);
          console.log("请求响应体信息", response.data);
        },
        (error) => {
          console.log("请求失败了", error.masage);
        }
      );
    },

    getSchoolCreateTime() {
      axios({
        // 请求url路径，可以写http://xxx.com/url注意跨越问题，也可以省略域名只写路径/url但是会在当前的服务中查找这个路径的接口
        // 避免浏览器的跨域，在vue.config中开启代理，
        // 注意vue.config的代理配置，当url请求的资源前端没有的时候会转发给对应的代理配置
        // 目前/api/V2前缀会代理到http://127.0.0.1:5002
        url: "/api/V2/url",
        // 设置请求的类型
        method: "POST",
        // 跟在url后面的参数设置
        params: {
          id: 666,
        },
        // 设置请求头
        headers: {
          "Content-Type": "application/json; charset=UTF-8",
          a: 888,
        },
        // 设置请求body
        data: {
          username: "王舞",
          password: "666666",
        },
      }).then(
        // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
        (response) => {
          console.log("请求响应状态码", response.status);
          console.log("请求响应状态字符串", response.statusText);
          console.log("请求响应全部header头信息", response.headers);
          console.log("请求响应体信息", response.data);
        },
        (error) => {
          console.log("请求失败了", error.masage);
        }
      );
    },
  },
};
</script>

<style scoped>
.school {
  background-color: aquamarine;
}
</style>
~~~