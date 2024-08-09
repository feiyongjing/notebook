### npm的依赖锁文件是package-lock.json，yarn的依赖锁文件是yarn.lock

### 根据项目的依赖锁文件来选择使用包管理工具，注意包管理不要混合使用
---

# 默认的node包管理工具，安装node就自带的包管理工具npm
~~~shell
# 空目录下初始化，进入交互式命令行来创建package.json文件，在交换式命令行中式文件内容
npm init 

# 生成的文件内容如下
{
  "name": "01-npm",    # 包的名称，无法使用大写字母和中文
  "version": "1.0.0",  # 包的版本，格式必须是x.x.x 大中小版本，版本号必须是数字
  "description": "npm初始化package.json",  # 包的描述
  "main": "index.js",  # 包的入口文件
  "scripts": {         # 设置脚本命令, 当使用npm run [脚本名称] 就可以执行对应的命令了，例如当前 npm run serve就是执行了以下的npm hello.js
    "test": "echo \"Error: no test specified\" && exit 1",  # 设置测试执行的命令
    "serve": "npm hello.js",
    "start": "npm app.js"   # start 比较特殊 可以简写为 npm start 执行脚本命令
  },
  "author": "",     # 作者
  "license": "ISC"  # 开源证书
}

# 快速初始，一些设置直接是默认的
npm init -y

# npn安装包查找网站，有点慢需要施法工具
# https://www.npmjs.com/

# 安装包，安装的包存放在node_modules文件夹下（开发依赖和生产依赖都安装在这里），
# 而package-lock.json文件是用于固定包的安装信息确保现在和将来安装的包版本一致
# 安装完包可以使用CommonJs引入包使用或者是ES6的模块化引入
# 在一些情况下依赖不网站需要 rm -rf node_modules package-lock.json 后重新安装依赖
npm install [包名]
# 简写
npm i [包名]
# 参数--save 简写-S 是用于安装包（依赖）在生产环境中，在package.json中的dependencies属性中看到
# 参数--save-dev 简写-D 是用于安装包（依赖）在开发环境中，在package.json中的devDependencies属性中可以看到
# 默认不加上述两个参数是安装生产依赖
# 安装指定版本的包 npm i [包名@版本号]

# 全局安装包，类似于安装命令，安装完后该命令就可以使用了
npm i -g [包名]

# 推荐安装 nodemon 可以用它代替node 命令启动程序，并且保存代码后自动热重启
# 安装nodemon后 可能无法使用nodemon启动nodejs程序，需要修改windows的策略或者使用其他终端执行nodemon test.js启动程序
# nodemon test.js

# 安装cnpm ,cnpm是淘宝的镜像库同步国外镜像库，用于快速安装包，cnpm的操作和npm的操作基本一致
# npm install -g cnpm --registry=https://registry.npmmirror.com
# 注意cnpm的安装需要于node的版本对应上

# 安装express-generator，这是express快速搭建项目
# npm i -g express-generator
# 生成代码到指定目录下，目录不存在会自动创建
# express -e [目录]

# npm设置淘宝镜像，注意这里是npm单独设置加速而不是cnpm，npm配置淘宝镜像后反而比cnpm快
# 清除缓存
npm cache clean --force
# 新淘宝镜像
npm config set registry https://registry.npmmirror.com
# cnpm设置新淘宝镜像
npm install -g cnpm --registry=https://registry.npmmirror.com

# 查看npm配置
npm config ls

# 查看全局安装包安装中那个目录
npm root -g 

# 安装项目的全部依赖，根据package.json和package-lock.json来安装项目的所有依赖
# 原因是一般情况下代码仓库都是不会存储node_modules文件夹下的依赖包
npm i 或者是 npm install

# 查看已安装包的信息
npm info [包名]

# 删除已经安装的包
npm remove [包名]
# 简写
npm r [包名]
# 删除全局安装包
npm r -g [包名]

# 运行package.json中的脚本
npm run [脚本名称]

~~~
 

# yarn管理js包
~~~shell
# 安装yarn
npm i -g yarn

# 查看yarn版本
yarn -v

# 安装生产依赖
yarn add [包名]

# 安装开发依赖
yarn add [包名] --dev

# 删除项目依赖包
yarn remove [包名]

# 安装全局包，注意安装的全局包无法直接使用命令，需要配置环境变量
yarn global add [包名]

# 查看全局包的安装位置，便于配置环境变量
yarn global bin

# 删除全局包
yarn global remove [包名]

# 安装项目全部依赖
# 在一些情况下依赖不网站需要 rm -rf node_modules yarn.json 后重新安装依赖
yarn

# 运行package.json中的脚本
yarn [脚本名称]

# 查看yarn的配置
yarn config list

# yarn 配置加速
yarn config set registry http://registry.npmmirror.com

# 注意yarn 的 依赖锁文件是yarn.lock

~~~

 

 

 

 

 

 

 

 

 