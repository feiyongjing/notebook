 
参考：https://nodejs.org/en
 
# node版本管理工具nvm  
参考 https://juejin.cn/post/7074108351524634655

~~~shell
# gitHub选择版本下载安装包  https://github.com/coreybutler/nvm-windows/releases

# 几个包如下，建议下载nvm-setup.zip
nvm-noinstall.zip： 这个是绿色免安装版本，但是使用之前需要配置
nvm-setup.zip：这是一个安装包，下载之后点击安装，无需配置就可以使用，方便。
Source code(zip)：zip压缩的源码
Sourc code(tar.gz)：tar.gz的源码，一般用于*nix系统

# nvm-setup.zip解压缩后运行nvm-setup.exe安装nvm
# 第一次选择目录：选择nvm的安装目录
# 第二次选择目录：选择node.js 的安装目录
# 第三次点击安装，在安装过程中会弹出：由于已经安装了 node，所以此时提示“你希望nvm管理已经安装的 node 版本吗”，点击 是待安装完成后测试是否安装成功

# 测试nvm是否安装成功
nvm -v

# 查看已经安装的所有nodejs版本
nvm ls 

# 可安装指定版本的nodejs
nvm install [node版本号]

# 切换到指定node版本
nvm use [node版本号]

# 卸载指定node版本
nvm uninstall [node版本号]

# 设置 nvm 存储node.js不同版本的目录 ,如果未设置，将使用当前目录。
nvm root [path] 

# 启用node.js版本管理
nvm on

# 禁用node.js版本管理(不卸载任何东西)
nvm off

# 设置用于下载的代理。留[url]空白，以查看当前的代理。设置[url]为none删除代理。
nvm proxy [url]

# 设置node镜像，例如 https://npm.taobao.org/mirrors/node/
nvm node_mirror [url]

# 设置npm镜像，例如 https://npm.taobao.org/mirrors/npm/
nvm npm_mirror [url]


~~~


 