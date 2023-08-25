### 由于浏览器不识别ES6的语法，使用打包工具可以将ES6的源码转换为浏览器认识的代码

### webpack是打包工具，打包编译源码输出编译好的文件，编译好的文件可以直接在浏览器中运行，同时还能压缩代码，让代码的运行和启动速度更快

# webpack的5大核心理念
~~~
# 1、entry(入口)
指定webpack从那个文件开始打包
# 2、output(输出)
指定webpack打包完的文件输出到哪里去，如何命名打包后的文件
# 3、loader(加载器)
webpack默认是只能处理js和json文件的，其他类型的资源文件需要借助于它们各自的loader，webpack才能解析打包资源
# 4、plugins（插件）
webpack的插件可以提供扩展功能
# 5、mode（模式）
主要是两种模式，development是开发模式，production是生产模式
~~~

 

 
# 安装使用
~~~shell
# webpack-cli是webpack的指令操作的命令行工具
npm i -D webpack webpack-cli typesctipt ts-loader
# webpack命令根据webpack.config.js配置文件直接打包，无需参数

# npx命令是将node_modules/.bin/目录下的文件添加为临时的环境变量，package.js的脚本是默认添加了npx的，所以可以省略npx
# 如果有webpack.config.js配置文件直接打包
npx webpack

# 以下命令没有webpack.config.js配置文件时通过命令行参数指定配置，webpack打包文件到当前目录下的dist文件夹下，默认dist文件夹不存在会创建
# --mode指定环境，development是开发模式，production是生产模式
# 使用开发模式打包不会压缩打包后的文件，而使用生产模式打包会压缩打包后的代码
# 默认webpack只能处理js文件打包，其他资源文件需要进行配置
npx webpack [需要打包的文件] --mode=development --config [指定的webpack配置文件]
npx webpack [需要打包的文件] --mode=production --config [指定的webpack配置文件]

# 安装css-loader来打包css资源，注意同时需要配置webpack.config.js配置文件
npm i -D style-loader css-loader
# 主要-D是安装在了开发环境

# 安装less-loader来打包less资源，注意同时需要配置webpack.config.js配置文件
# 如果已经安装了style-loader css-loader不必重复安装，只需要安装less less-loader
npm i -D style-loader css-loader less less-loader
# 主要-D是安装在了开发环境

# 安装sass-loader来打包sass资源和scss资源，注意同时需要配置webpack.config.js配置文件
# 如果已经安装了style-loader css-loader不必重复安装，只需要安装sass sass-loader
npm i -D style-loader css-loader sass sass-loader
# 主要-D是安装在了开发环境

# 安装stylus-loader来打包styl资源，注意同时需要配置webpack.config.js配置文件
# 如果已经安装了style-loader css-loader不必重复安装，只需要安装stylus-loader
npm i -D style-loader css-loader stylus stylus-loader
# 主要-D是安装在了开发环境

# 使用style-loader会出现闪屏现象，有些样式的加载在网速加载慢的时候会出现html已经显示出来了但是样式没有显示出来
# 使用mini-css-extract-plugin替换style-loader可以避免闪屏现象，插件封装样式，将全部的css打包成一个css文件
# 注意同时需要配置webpack.config.js配置文件插件设置以及使用MiniCssExtractPlugin.loader替换stylus-loader
npm i -D mini-css-extract-plugin
# 配置如下
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  module: {
    rules: [
      {
        test: /.s?css$/,
        use: [
            MiniCssExtractPlugin.loader,   // 使用MiniCssExtractPlugin.loader替换stylus-loader
            "css-loader", 
            "less-loader"
        ],
      },
    ],
  },
  
  optimization: {
    // 开发环境下启用css打包成单独的文件
    minimize: true,
  },
  
  plugins: [
    // 使用MiniCssExtractPlugin.loader替换stylus-loader
    new MiniCssExtractPlugin({
    // 将全部的css打包成一个css文件
    filename:'css/main.css'
    }),
  ],
};


# 压缩打包后的样式文件
npm i -D css-minimizer-webpack-plugin
# 配置如下
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
module.exports = {
  plugins: [new MiniCssExtractPlugin()],
};

# postcss-loader处理样式的兼容性
npm i -D postcss-loader postcss postcss-preset-env
# 配置如下
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',  // postcss-loader处理样式的兼容性，注意需要写在css-loader的前面，其他样式的后面
            options: {
              postcssOptions: {
                plugins: [
                  [
                    'postcss-preset-env',  // postcss-preset-env可以解决大多数的样式兼容性问题
                  ],
                ],
              },
            },
          },
        ],
      },
    ],
  },
};


# 安装html-webpack-plugin来打包html资源，默认在production生产模式下会压缩打包文件 
# 注意同时需要配置webpack.config.js配置文件
npm i -D html-webpack-plugin
# 主要-D是安装在了开发环境

# 安装配置webpack-dev-server进行热启动，只要一修改源码，webpack就会自动打包编译
npm i -D webpack-dev-server
# 配置webpack.config.js配置文件
module.exports = {
  // 开发热启动，不会生产打包后的资源文件，而是在直接打包在内存中运行
    devServer: {
        host: "localhost",  //启动服务器ip
        port: "3000",       //启动服务器的端口
        open: true          //是否自动打开浏览器
    },
  // ...
};
# 热启动
npx webpack serve
# 不设置配置文件使用命令传递参数热启动
webpack serve --mode development --open chrome.exe 
# mode是启动环境设置，production生产环境，development开发环境
# open chrome.exe 是在chrome浏览器运行 

# webpack配置sourceMap来在编译打包后生成map文件存储源代码和打包后的代码行和列的映射关系，编译代码出现问题后快速定位到源代码
# 如下是开发环境和生产环境的配置
# 开发配置
module.exports = {
    mode: 'development',
    // 配置cheap-module-source-map来在编译打包后生成map文件存储源代码和打包后的代码行和列的映射关系，编译代码出现问题后快速定位到源代码
    // 注意cheap-module-source-map用于开发环境，不能用于生产环境
    devtool: 'cheap-module-source-map',
}
# 生产配置
module.exports = {
     mode: 'production',
    // 配置sourceMap来在编译打包后生成map文件存储源代码和打包后的代码行和列的映射关系，编译代码出现问题后快速定位到源代码
    // 注意source-map用于生产环境下
    devtool:'source-map',
}

# 多进程打包js和eslint检查，webpack配置见详细的配置文件
npm i -D thread-loader

# 图片压缩优化，无损优化图片，webpack配置见详细的配置文件
npm i -D imagemin image-minimizer-webpack-plugin
npm i -D imagemin-gifsicle imagemin-jpegtran imagemin-optipng imagemin-svgo 


npm i -D typesctipt ts-loader


~~~ 
---


# webpack的核心配置文件webpack.config.js
~~~javaScript
const os = require('os')
const path = require('path')
const ESLintPlugin = require('eslint-webpack-plugin');    // 检查代码规范插件
const HtmlWebpackPlugin = require('html-webpack-plugin'); // html打包插件
const MiniCssExtractPlugin = require("mini-css-extract-plugin");  // 插件封装样式css打包成一个文件
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin"); // 插件压缩打包的css文件
const TerserWebpackPlugin = require("terser-webpack-plugin");  // 默认是使用该插件单线程压缩js代码，如果需要多线程压缩需要进行配置
const ImageMinimizerPlugin = require("image-minimizer-webpack-plugin"); // 插件无损优化压缩图片

// 获取系统cpu的核数，基于该数字启动对应数量的线程同时进行webpack打包
const threads = os.cpus().length;

// 获取不同样式的loader配置
function getStyleLoader(pre) {
    return [
        // 检测使用的loader，顺序是自下而上依次使用loader检测
        // 'style-loader',  // 将js中css通过style标签添加到html文件中生效，所以打包后不会出现.css文件，而是在浏览器中出现style标签
        MiniCssExtractPlugin.loader,   // 使用MiniCssExtractPlugin.loader替换stylus-loader
        'css-loader',     // 将css资源编译成commonjs的模块到js中，前提是js文件中引用了css文件
        {
            loader: 'postcss-loader',  // postcss-loader处理样式的兼容性，注意需要写在css-loader的前面，其他样式的后面
            options: {
                postcssOptions: {
                    plugins: [
                        [
                            'postcss-preset-env',  // postcss-preset-env可以解决大多数的样式兼容性问题
                        ],
                    ],
                },
            },
        },
        pre,  // 不同样式的loader
    ].filter(Boolean) // filter(Boolean)过滤空的元素
}


module.exports = {
    // 单入口，打包输出一个js文件
    entry: './src/main.js',
    // 多文件入口，打包输出多个js文件
    // entry: {
    //     main: "./src/main.js",
    //     sum: "./src/js/sum.js",
    //     count: "./src/js/count.js",
    // },
    // 输出
    output: {
        // 打包后的文件输出目录，需要使用绝对路径
        path: path.resolve(__dirname, '../dist'),
        // 打包后的文件名称，[name表示原文件名称]
        filename: '[name].js',
        // 自动清空上次打包的内容
        // 在打包前自动清空path指定的打包目录下全部内容，再打包输出到path目录下
        clean: true,
    },

    // 加载器
    module: {
        // 每个loader的配置
        rules: [
            {
                // onnOf代表每个文件只能被一个loader处理，避免重复的loader检查文件是否可以处理，提高weback打包速度
                oneOf: [
                    {
                        test: /\.css$/,  //css-loader只检测.css后缀的文件
                        use: getStyleLoader()
                    },
                    {
                        test: /\.less$/,  //less-loader只检测.less后缀的文件
                        use: getStyleLoader('less-loader') // 将less资源编译成css文件，前提是js文件中引用了less文件
                    },
                    {
                        test: /\.s[ac]ss$/,  //sass-loader只检测.sass和.scss后缀的文件
                        use: getStyleLoader('sass-loader') // 将sass资源或scss资源编译成css文件，前提是js文件中引用了sass文件或scss文件
                    },
                    {
                        test: /\.styl$/,  //stylus-loader只检测.styl后缀的文件
                        use: getStyleLoader('stylus-loader') // 将styl资源编译成css文件，前提是js文件中引用了styl文件
                    },
                    {
                        test: /\.(png|jpg|gif|webp|svg)$/,  //只检测对应后缀的图片文件
                        type: "asset", //asset 在导出一个 data URI 和发送一个单独的文件之间自动选择。
                        //解析
                        parser: {
                            //转base64的条件
                            dataUrlCondition: {
                                //图片小于50kb,就会被base64处理，一般处理8-12KB的图片
                                //优点： 减少请求数量
                                //缺点： 图片体积会变大
                                maxSize: 50 * 1024, // 25kb
                            }
                        },
                        generator: {
                            //输出图片的路径图片名称取前10位hash值
                            filename: 'img/[hash:10][ext][query]',
                        },
                    },
                    {
                        test: /\.(ttf|woff|woff2?|mp3|mp4|avi)$/,  //只检测对应后缀的视频、音频、字体图片等文件
                        type: "asset/resource", //asset/resource 打包输出原文件，不会加密
                        generator: {
                            //输出图片的路径图片名称取前10位hash值
                            filename: 'media/[hash:10][ext][query]',
                        },
                    },
                    {
                        test: /\.m?js$/,  // babel设置只处理js后缀文件
                        // include和exclude两个配置只能写一个
                        // include: path.resolve(__dirname,"../src/"), // 只处理src目录下的文件，其他文件babel都不处理
                        exclude: /(node_modules)/, // 排除node_modules文件不处理，而其他文件babel都处理
                        use: [
                            {
                                loader: 'thread-loader',  // thread-loader开启多线程处理babel编译
                                options: {
                                    works: threads,   //  设置线程数量  
                                }
                            },
                            {
                                loader: 'babel-loader',
                                options: {
                                    cacheDirectory: true, // 开启babel缓存，缓存打包的文件，在webpack打包时没有修改的js代码从缓存取无需二次打包
                                    cacheCompression: false // 关闭缓存文件压缩
                                }
                            }
                        ],
                    }
                ]

            }

        ],


    },
    // 插件使用，注意插件需要使用模块化引入
    plugins: [
        // eslint插件检查js和JSX
        new ESLintPlugin({
            // 检查哪些文件的代码，如下是检查当前目录下的src目录下的所有文件
            context: path.resolve(__dirname, '../src'),
            // 排除检查哪些目录下的文件代码检查，默认是排除node_modules的依赖文件代码检查
            exclude: "node_modules",
            // 开启缓存，在webpack打包检查代码时只根据缓存与源码比较只检查有修改文件代码的检查，提高打包速度
            cache: true,
            // 缓存存放目录
            cacheLocation: path.resolve(
                __dirname,
                "../node_modules/.cache/eslintcache"
            ),
            // 开启多线程处理eslint插件检查js和JSX
            threads: threads,
        }),

        // html打包插件，默认在production生产模式下会压缩打包文件 
        new HtmlWebpackPlugin({
            // 模板：以指定文件创建新的文件
            // 新的html文件特点：1、结构和原来的一致 2、会自动引入打包输出的资源
            template: path.resolve(__dirname, '../public/index.html')
        }),

        // 使用MiniCssExtractPlugin.loader替换stylus-loader
        new MiniCssExtractPlugin({
            // 将全部的css打包成一个css文件
            filename: 'css/main.css'
        }),

        // 插件无损优化压缩图片
        new ImageMinimizerPlugin({
            minimizer: {
                implementation: ImageMinimizerPlugin.imageminGenerate,
                options: {
                    plugins: [
                        ["gifsicle", { interlaced: true }],
                        ["jpegtran", { progressive: true }],
                        ["optipng", { optimizationLevel: 5 }],

                        [
                            "svgo",
                            {
                                plugins: [
                                    "preset-default",
                                    "prefixIds",
                                    {
                                        name: "sortAttrs",
                                        params: {
                                            xmlnsOrder: "alphabetical",
                                        },
                                    },
                                ],
                            },
                        ],
                    ],
                }
            },
        }),

    ],

    // 开发热启动，不会生产打包后的资源文件，而是在直接打包在内存中运行
    devServer: {
        host: "localhost",  //启动服务器ip
        port: "3000",       //启动服务器的端口
        open: true          //是否自动打开浏览器
    },


    optimization: {
        // 开发环境下启用css打包成单独的文件
        minimize: true,

        minimizer: [
            // 插件压缩打包的css文件
            new CssMinimizerPlugin(),

            // 配置多线程加快webpack压缩js代码打包
            new TerserWebpackPlugin({
                parallel: threads
            })
        ],

        // 代码分割配置
        splitChunks: {
            // 单文件入口只需要配置chunks: "all"，多文件入口需要配置组
            chunks: "all",
            // 以下是默认值
            // minSize:20000,  // 分割代码最小的大小,默认是20kb
            // minRemainingSize: 0, // 确保提取的文件大小的最小值为0
            // minChunks:1,//提取的文件至少被引用的次数
            // maxAsyncRequests:10, // 按需加载时并行加载的文件的最大数量
            // maxInitialRequests: 30, //入口js文件最大并行请求数量
            // enforceSizeThreshold: 50000, // 超过50kb一定会单独打包（此时会忽略minRemainingSize、maxAsyncRequests、maxInitialRequests）
            // cacheGroups:{  // 组，哪些模块要打包到一个组
            //     defaultVendors:{  //组名
            //         test:/[\\/]node_modules[\\/]/, //需要打包到一起的模块
            //         priority: -10, //权重（越大越高）
            //         reuseExistingChunk:true,//如果当前chunk包含已从bundle中拆分出的模块，则它将被重用，而不是生成新的模块
            //     },
            //     default:{ //其他没有写的配置会使用上面的默认值
            //         minChunk:2, //提取的文件至少被引用的次数
            //         priority: -20, //权重（越大越高）
            //         reuseExistingChunk:true,//如果当前chunk包含已从bundle中拆分出的模块，则它将被重用，而不是生成新的模块
            //     },
            // },
            // 修改配置
            cacheGroups: {  // 组，哪些模块要打包到一个组
                default: { //其他没有写的配置会使用上面的默认值
                    minSize:0, // 分割代码最小的大小
                    minChunks: 2, //提取的文件至少被引用的次数
                    priority: -20, //权重（越大越高）
                    reuseExistingChunk: true,//如果当前chunk包含已从bundle中拆分出的模块，则它将被重用，而不是生成新的模块
                },
            },
        }

    },

    // 模式 development是开发模式，production是生产模式 
    // 使用开发模式打包不会压缩打包后的文件，而使用生产模式打包会压缩打包后的代码
    mode: 'development',
    // 配置cheap-module-source-map来在编译打包后生成map文件存储源代码和打包后的代码行和列的映射关系，编译代码出现问题后快速定位到源代码
    // 注意cheap-module-source-map用于开发环境，不能用于生产环境
    devtool: 'cheap-module-source-map'
}
~~~
 

 