# react项目打包运行

### 项目打包和简单运行启动
- 真实的项目一般是将打包好的build或者dist交给运维人员去配置nginx服务进行转发
~~~shell
# 打包完成之后生成build文件夹
npm run build

# 安装sever，快速搭建一个简易的静态服务
npm i -g serve

# 进入build文件夹运行serve，通过-s参数指定build文件夹运行服务
cd build
serve -s build
~~~


