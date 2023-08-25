
~~~
# 需要提前安装node环境
# 全局安装TypeScript
npm i -g typescript

# ts文件可以编译成任意版本的js文件
# 编译ts文件，编译好之后会在当前目录下出现test.js文件，默认是编译成es3的文件
tsc test.ts
# -w参数 表示watch监视，发现ts文件发生变化就自动编译，注意发现过程可能有一定的延迟

# 如果需要监视整个项目的ts文件那么需要配置tsconfig.json文件，然后直接tsc 无需添加-w参数就可以监视ts文件变动自动编译
{
    // tsconfig.json是ts编译器的配置文件，ts编译器可以根据它的信息来对代码进行编译

    // include是设置那些目录下的文件需要编译，**表示路径下的全部目录，*表示路径下的全部文件
    "include": [
        "./src/**/*"  
    ],
    // exclude是设置那些目录下的文件不需要编译，**表示路径下的全部目录，*表示路径下的全部文件
    "exclude": [
        "./src/hello/**/*"
    ],
    // 与include类似，但是只能指定具体的文件编译
    // "files": []

    // extends表示当前文件会自动包含指定json文件的所有配置文件
    // "extends":"./config/base"

    // 编译器的选项
    "compilerOptions": {
        // target指定编译后的js代码版本，仅限于语法的转换一些API无法转换
        "target": "ES3",
        // module指定编译后的js是哪种模块化，一般是commonjs和 ES2015（ES6） 
        "module": "ES2015",
        // lib是指定需要依赖的库，在编译时会按照这些库来检查代码，如果代码使用了lib没有指定的库会直接报错
        // 一般情况下web前端开发是不会配置lib的，除非是nodejs开发才会用到
        // "lib": []

        // outDir:指定编译后的js文件放在那个目录，默认是和原ts文件放在一起的
        "outDir": "./dist",

        // 将编译后的全部代码合并到一个指定的文件中
        // 注意如果使用了模块化需要修改module的值，只有特定的模块化才能合并模块化的代码
        // "outFile": "./dist/app1.js",

        // allowJS是否对项目下的js文件编译，默认是false
        // "allowJs": false,
        // checkJS是否对项目下的js文件按照ts的语法检查，默认是false
        // "checkJs": false

        // removeComments表示是否去除编译后的js文件注释是否去除，默认是false
        // "removeComments": false

        // noEmit表示是否不生成编译后的js文件，默认是false
        // 注意虽然没有生成编译后的js文件，但是编译ts文件时的语法检查还是有的
        // "noEmit": false

        // noEmitOnError当编译有错误时是否不生成编译的js文件，默认是false
        "noEmitOnError": true,

        // 所有严格检查ts代码的总开关，默认是false
        // 下面的几个都是属于代码检查，还有更多请自行查看文档
        "strict": true,

        // alwaysStrict设置编译后的文件是否使用严格模式，默认是false，
        // 严格模式的js文件开头是"use strict"，当然如果js文件已经引入了模块那么相当于已经进入了严格模式
        "alwaysStrict": true,

        // noImplicitAny设置是否禁止使用隐式的any类型，默认是false，
        "noImplicitAny": true,


        // 是否禁止使用类型不明确的this，默认是false
        "noImplicitThis": true,

        // 是否检查空值引用（对一个null调用函数和属性） ，默认是false
        "strictNullChecks": true

    }
}

~~~