# react脚手架创建项目
~~~shell
# 创建项目
npx create-react-app react-my-app
# 如果npx找不到就执行npm install -g npx先安装npx, 注意npx是需要npm的版本在5.2以上
# 也可以使用npm安装npm install -g create-react-app 再 npm init react-app react-my-app 创建项目
# react-my-app是项目名称

# 进入项目目录
cd react-my-app

# 启动项目，会在本地3000开启服务
npm start

~~~

# 初始化之后的项目结构
~~~
my-app
├── README.md
├── node_modules          // 依赖包存放目录，在代码管理仓库中没有这个目录
├── package.json          // 依赖包配置文件
├── .gitignore            // git排查文件版本追踪设置
├── public                // 静态资源文件夹
│   ├── favicon.ico       // 网站页签图标
│   ├── index.html        // 主页面(重要文件)
│   ├── logo192.png       // logo图
│   ├── logo512.png       // logo图
│   ├── manifest.json     // 应用加壳的配置文件
│   └── robots.txt        // 爬虫协议文件
└── src                   // 主要源码文件
    ├── App.css           // APP组件CSS样式
    ├── App.js            // APP组件(重要文件)
    ├── App.test.js       // APP组件测试
    ├── index.css         // 主页面样式
    ├── index.js          // 项目的入口文件（重要文件，类似于main函数）
    ├── logo.svg          // logo图
    ├── serviceWorker.js  // 记录页面的性能
    └── setupTests.js     // 项目的整体组件测试
~~~

# public/index.html 主页面例子如下
~~~html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <!-- 引入页签图标, %PUBLIC_URL%代表public文件夹路径-->
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />

    <!-- 开启理想视口,用于做移动端网页的适配 -->
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!-- 用于配置浏览器页签和地址栏的颜色(仅支持安卓手机浏览器) -->
    <meta name="theme-color" content="#000000" />

    <!-- 描述网站的元信息 -->
    <meta
      name="description"
      content="Web site created using create-react-app"
    />

    <!-- 设置网页添加到手机屏幕后的图标(仅支持苹果手机) -->
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />

    <!-- 引用加壳配置文件,即项目外套个壳变成安卓app应用或者是IOS的app应用
         配置文件中主要写一些app的名称、图标、需要访问手机的权限（相册、麦克风、摄像头）等
     -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <!-- 浏览器不支持JavaScript就显示这句话 -->
    <noscript>You need to enable JavaScript to run this app.</noscript>

    <!-- 组件容器 -->
    <div id="root"></div>
  </body>
</html>
~~~

# src/index.js 项目的入口文件（类似于main函数）例子如下
~~~js
import React from 'react';                       // react的核心库
import ReactDOM from 'react-dom/client';         // 提供操作DOM的react库
import './index.css';                            // 主页面的样式
import App from './App';                         // APP组件引入
import reportWebVitals from './reportWebVitals'; // 引入记录页面的性能

// 获取root容器节点
const root = ReactDOM.createRoot(document.getElementById('root'));

// 解析组件标签，找到组件，生成虚拟DOM转化为真实DOM在root容器节点中渲染
// React.StrictMode会检查组件和它的后辈组件是否合理（其实是react的API是否适用）
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// 记录页面的性能
reportWebVitals();
~~~

# src/App.js APP组件例子如下
~~~js
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
~~~



