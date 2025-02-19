### pip是python包的通用管理器，而conda是一个与语言无关的跨平台环境管理器
- 安装参考：https://blog.csdn.net/fan18317517352/article/details/123035625

- 使用参考：https://zhuanlan.zhihu.com/p/32925500

- Anaconda Prompt 命令行窗口打开参考：https://blog.csdn.net/weixin_35684521/article/details/123036558

- Anaconda Navigator 图形操作参考：https://www.zhihu.com/question/58033789/answer/254673663?utm_source=wechat_session&utm_medium=social

- Anaconda 卸载参考：https://blog.csdn.net/Lord_Bao/article/details/114170382

# 管理环境

### Windows用户没有配置环境变量请打开“Anaconda Prompt”，可以直接使用conda命令。如果使用其他终端操作需要配置环境变量
### macOS和Linux用户请先设置环境变量，再打开“Terminal”（“终端”）进行操作

~~~shell
# 创建新环境，例如conda create -n Python312 python=3.12
conda create --name [env_name] [package_names]
# --name env_name 指定创建的环境名，--name 可以简写为 -n
# 默认该环境的目录是当前目录，可以使用 --prefix 指定环境目录，但是使用--prefix就无法使用--name指定环境名称，建议还是切换到预先创建的环境目录然后创建环境时使用--name指定环境名称
# package_names 即安装在环境中的包名，如果要安装指定的版本号，则只需要在包名后面以 = 和版本号的形式执行
# 如果创建环境后安装Python时没有指定Python的版本，那么将会安装与Anaconda版本相同的Python版本
# 当成功创建或切换环境之后，在该行行首将以 (env_name) 或 [env_name] 开头。其中， env_name 为当前的环境名
# 退出当前环境后行首的环境表示将消失

# 保存当前环境安装的包清单列表到指定文件
conda env export > [environment.yml]
# 切换环境或退出环境之前必须保存包清单列表，否则下一次进入环境将会丢失已经安装的包

# 根据包清单列表加载环境
conda env create -f [environment.yaml]

# 复制环境
conda create --name [new_env_name] --clone [old_env_name]
# old_env_name 是旧的环境名称或者是环境路径

# 激活或切换环境
source activate [env_name]
# env_name是环境名称或者是环境路径
# Windows下Git Bash 正常使用上述命令
# Windows下直接使用activate [env_name] 切换环境

# 退出环境
source deactivate
# Windows下Git Bash 正常使用上述命令
# Windows下cmd 使用 conda deactivate 退出环境

# 列出当前环境下安装的包
conda list

# 在指定环境下安装包，如果不指定就是在当前环境下安装包
conda install --name [env_name] [package_name]
conda run -n [env_name] pip install [package_name]
# env_name是环境名称或者是环境路径
# 找不到请去 https://anaconda.org 搜索需要的包
# 注意有些包如果找不到也可以使用 pip 安装包

# 卸载指定环境中的包，如果不指定就是卸载当前环境的包
conda remove --name [env_name] [package_name]
# # env_name是环境名称或者是环境路径

# 更新指定包
conda update [package_name]
conda upgrade [package_name]
conda update conda #升级conda
conda update anaconda #升级anaconda前要先升级conda
conda update --all #升级所有包

# 删除环境，根据环境名称或者是环境路径删除环境，注意谨慎使用环境路径删除环境，这可能删除该路径和上层路径中的文件
conda remove --name [env_name] --all
conda env remove --prefix [env_path]
# env_name 是需要删除的环境名称或者是环境路径

conda clean -p #删除没有用的包
conda clean -t #删除保存下来的压缩文件（.tar）

# 以下三条命令都可以显示已经存在的环境列表
conda info --envs
conda info -e
conda env list
# 会列出环境名称和环境创建的目录，* 代表当前所在的环境
~~~
---