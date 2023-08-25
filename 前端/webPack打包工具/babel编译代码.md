
~~~shell
# 安装babel来编译javaScript、typescript、JSX(React是使用JSX)代码，从浏览器不支持的es6等高版本编译成支持的低版本代码
# 注意webpack会默认添加一些箭头函数，所有浏览器可能还是无法运行代码
# 代码规范检查配置文件是项目根目录下的 .babelrc 或 .babelrc.js 或 .babelrc.json 或 babel.config.js 或 babel.config.json 之一，注意只需要配置其中一个就行了
# 注意同时需要配置webpack.config.js配置文件
npm i -D babel-loader @babel/core @babel/preset-env 
# 主要-D是安装在了开发环境

# Babel 对一些公共方法使用了非常小的辅助代码，默认情况下会被添加到每一个需要它的文件中。
# 使用babel/plugin-transform-runtime来避免重复引入。
npm install -D @babel/plugin-transform-runtime
~~~
 

### babel.config.js配置文件

~~~javaScript
module.exports = {
    // @babel/preset-env是智能预设，能编译ES6的语法
    // @babel/preset-react 能编译React jsx语法的预设
    // @babel/preset-typescript 能编译TypeScript语法的预设
    presets:["@babel/preset-env"],
    plugins: ['@babel/plugin-transform-runtime']
}
~~~
 

### webpack配置

~~~javaScript
module.exports ={
    module: {
        rules: [
            {
                test: /\.m?js$/,  // babel设置只处理js后缀文件
                exclude: /(node_modules|bower_components)/,  // 排除node_modules和bower_components文件不处理
                use: {
                    loader: 'babel-loader',
                }
            }
        ]
    }
}
~~~
 