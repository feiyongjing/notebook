# 修改vue.config.js配置后端服务的代理，例子如下
~~~js
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  lintOnSave: false, //关闭语法检测，一些创建后没有使用的函数或者变量会导致语法检查不过
  transpileDependencies: true,
  // 开启代理，避免浏览器的跨域
  // devServer: {
  //   // 当请求的资源前端没有的时候会转发给配置的http://127.0.0.1:5000
  //   // 注意这种只能配置一个代理，并且只有前端没有的资源才会走代理
  //   proxy: "http://127.0.0.1:5000"
  // },

  // 开启多代理，避免浏览器的跨域
  // 请求的资源前端没有的时候会转发给配置的后端服务器
  devServer: {
    proxy: {

      // 代理域名后面以/xxx为前缀的请求，
      // 默认请求后端路径会带上这个前缀
      // 例如请求 http://sss.com/xxx/getUser 最终访问的后端是http://127.0.0.1:5000/xxx/getUser
      // 如果代理转发想要去除请配置pathRewrite属性
      '/xxx': {
        // 目标后端服务器
        target: "http://127.0.0.1:5000",
      },


      // 代理域名后面以/api/V1为前缀的请求，
      // 默认请求后端路径会带上这个前缀，如果代理转发想要去除请配置pathRewrite属性
      '/api/V1': {
        // 目标后端服务器
        target: "http://127.0.0.1:5001",
        // 使用正则表达式匹配对应的路径前缀，将前缀替换为空字符串实现去除前缀
        // 例如请求 http://xxx.com/api/V1/getUser 最终访问的是http://127.0.0.1:5001/getUser
        pathRewrite: { '^/api/V1': '' },
        // ws是用于支持websocket，默认开启，一般情况下默认
        ws: true,
        // changOrigin是用于控制向后端发起请求的请求头中host的值，
        // true代表请求头host的值是后端服务器的域名ip，false代表请求头host的值是前端服务器的域名ip
        // 默认是true，一般情况下默认避免后端服务器做限制导致请求被拦截
        changOrigin: true
      },

      // 代理域名后面以/api/V2为前缀的请求
      '/api/V2': {
        // 目标后端服务器
        target: "http://127.0.0.1:5002",
        pathRewrite: { '^/api/V2': '' },
        ws: true,
        changOrigin: true
      }
    }
  },

})
~~~





