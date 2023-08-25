
~~~shell
# 安装vue-cli
npm install -g @vue/cli

# 将npm设置为淘宝镜像
npm config set registry https://registry.npm.taobao.org

# windows下git-bash创建项目，执行会让你选择使用vue2还是vue3创建项目
winpty vue.cmd create 项目名
# windows的cmd 下创建项目，执行会让你选择使用vue2还是vue3创建项目
vue create 项目名

# 进入创建的项目
cd 项目名

# 启动项目，默认是8080端口
npm run serve

# 如果需要自定义脚手架请在package.json文件同级目录下创建vue.config.js文件
# vue.config.js文件配置参考https://cli.vuejs.org/zh/config/
# 在修改vue.config.js文件配置后注意重新npm run serve启动项目

~~~