# react配置代理

## 方式一：在package.json中配置
- 所有访问前端服务监听端口的请求都先在前端资源中查找是否存在，存在就直接返回，不存在就去配置的proxy代理指定的服务请求查找返回
- 缺点是只能配置一个代理后端服务
~~~json
{
  "name": "react-my-app",
  "version": "0.1.0",
  "private": true,
  ...,
  "proxy": "http://localhost:5000"  
}
~~~

## 方式二：src/setupProxy.js 配置
- 参考 https://developer.aliyun.com/article/1397146
- 在 src 中，查找setupProxy.js进行编辑，没有就新建 （文件名固定，不可自定义，react 脚手架会识别）
- npm info http-proxy-middleware 检查http-proxy-middleware版本，1.0 以后是如下配置
~~~js
const proxy = require('http-proxy-middleware')
module.exports = function (app)  {
  app.use(
    // 代理 1
    proxy.createProxyMiddleware('/api', {             // 匹配到 '/api' 前缀的请求，就会触发该代理配置
      target: 'http://127.0.0.1:6000/api',            // 请求转发地址
      changeOrigin: true,                             // 是否跨域
      /*
        changeOrigin 为 true 时，后端服务收到的请求头中的host为：127.0.0.1:6000，也就是后端服务地址的 host
        changeOrigin 为 false 时，后端服务收到的请求头中的host为：localhost:3000，也就是后端服务地址的 host
        changeOrigin 默认 false，但一般需要将 changeOrigin 值设为 true，根据自己需求调整
      */
      pathRewrite: {
        '^/api': ''                                   // 重写请求路径，即前缀路径 /api 替换为指定的空字符串
      }
    }),
    // 代理 2，为什么 2 写前面，因为匹配规则，上面第一个已经是 /api 了，要不然会优先匹配到第一个代理上
    proxy.createProxyMiddleware('/2api', {
      target: 'http://127.0.0.1:6000/api',
      changeOrigin: true,
      pathRewrite: {
        '^/2api': ''
      }
    }),
    // 代理 3，这种写法是最规范的，前后都加 /
    proxy.createProxyMiddleware('/3api/', {
      target: 'http://127.0.0.1:6000/api/',
      changeOrigin: true,
      pathRewrite: {
        '^/3api/': ''
      }
    }),
    // 代理 4，这种代理标识尾部加 / ，代理地址尾部没有
    proxy.createProxyMiddleware('/4api/', {
      target: 'http://127.0.0.1:6000/api',
      changeOrigin: true,
      pathRewrite: {
        // '^/4api/': ''  // 这种替换成空，也没问题，但是不严谨
        '^/4api/': '/'    // 这样写更规范
      }
    })
    // ..... 多个代理
  )
}
~~~


