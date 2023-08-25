
~~~shell
# 安装eslint来打包检查javaScript和JSX(React是使用JSX)资源的代码规范
# 代码规范检查配置文件是项目根目录下的 .eslintrc 或 .eslintrc.js 或 .eslintrc.json 之一，注意只需要配置其中一个就行了
# .eslintingore文件配置需要忽略eslint检查的目录文件
# 注意同时需要配置webpack.config.js配置文件
# 如果已经安装了eslint不必重复安装，只需要安装eslint-webpack-plugin
npm i -D eslint eslint-webpack-plugin
# 主要-D是安装在了开发环境


npm i -D babel-eslint
~~~
 

### .eslintrc.js配置文件
~~~javaScript
module.exports = {
    // 继承 eslint规则，不同框架的项目或者是不同的企业有不同的代码规范
    // 下面rules配置的规则会覆盖继承的eslint规则检查
    extends: ["eslint:recommended"],

    env: {
        node: true,    // 启用node中的全局变量
        browser: true, // 启用浏览器中的全局变量
    },

    parserOptions: {
        ecmaVersion: 6,     // 使用es6的检查
        sourceType: "module", // 使用es的模块化检查
    },
    rules: {
        // 配置是语法检查项和检查规则，
        // 'off'或0是关闭规则检查
        // 'warn'或1是开启规则检查，使用警告级别的错误，不会导致程序的退出
        // 'error'或2是开启规则检查，使用错误级别的错误，当触发时会导致程序的退出
        "no-var": 2,// 不能使用var定义变量
    }
}
~~~
 

### webpack配置eslint插件
~~~javaScript
const ESLintPlugin = require('eslint-webpack-plugin');

module.exports = {
  // ...
  plugins: [
      new ESLintPlugin({
        // 检查哪些文件的代码，如下是检查当前目录下的src目录下的所有文件
        context: path.resolve(__dirname, 'src'),
      })],
  // ...
};
~~~
 

 

 

 