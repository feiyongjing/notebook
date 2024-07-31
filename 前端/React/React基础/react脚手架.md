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

# 项目结构
~~~
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    ├── serviceWorker.js
    └── setupTests.js
~~~




