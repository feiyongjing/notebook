### 安装poetry
~~~shell
pip install poetry
~~~

### 初始化 poetry 项目
~~~shell
# 创建项目目录并且进入
mkdir poetry-demo
cd poetry-demo

# 初始化 poetry 项目
poetry init
# 会跳出来一连串的互动对话，用于创建项目的配置文件，这里直接全部一路回车，然后看一下生成的 pyproject.toml 配置文件，例子如下
# [tool.poetry]                                      # 是最基本的section，然后它由多个 sections 组成
# name = "poetry-demo"                               # 项目名称
# version = "0.1.0"                                  # 项目版本
# description = ""                                   # 项目描述
# authors = ["zhengxinonly <pyxxponly@gmail.com>"]   # 项目作者和邮箱地址
# readme = "README.md"                               # 项目的README.md文件位置           
# 
# 
# [tool.poetry.dependencies]                         # 默认情况下，poetry 会从 Pypi 库中查找依赖项，只需要写名称、版本就行了
# python = "^3.10"                                   # 重点：必须声明与依赖包兼容的python版本
#  
#  
# [build-system]
# requires = ["poetry-core"]
# build-backend = "poetry.core.masonry.api"

# 或者直接创建一个包含基本项目结构的新项目
poetry new [项目目录]
~~~

### 虚拟环境管理
~~~shell
# poetry虚拟环境全局配置查看
poetry config --list 

# 配置 poetry 将虚拟环境创建在项目目录中，而不是默认的全局缓存目录。这对于确保项目的隔离性和可移植性非常有用。
poetry config virtualenvs.in-project true
# 创建项目专属的虚拟环境：在项目目录中创建和管理虚拟环境，虚拟环境将位于项目目录的 .venv 文件夹中。
# 增强项目的可移植性：当项目与其虚拟环境一起复制或移动到其他地方时，不需要重新创建虚拟环境或安装依赖项。
# 便于版本控制：可以更容易地将项目的虚拟环境添加到版本控制中，尽管通常 .venv 文件夹会被添加到 .gitignore 文件中以避免版本控制虚拟环境。
# Windows激活已有的虚拟环境.\.venv\Scripts\activate
# macOS 和 Linux 激活已有的虚拟环境 source .venv/bin/activate
# 验证虚拟环境激活which python 输出应指向项目目录中的 .venv 文件夹，例如 /path/to/your/project/.venv/bin/python

# 使用指定的 Python 解释器创建一个新的虚拟环境，虚拟环境的名称是 项目名称-pypython版本，例如 your-project-py3.8
poetry env use [Python解释器(可以是配置好的path环境变量的Python解释器也可以是绝对路径指定的Python解释器)]

# 查看所有虚拟环境
poetry env list
# --full-path参数显示虚拟环境的路径

# 激活虚拟环境，会检查当前目录或上层目录是否存在 pyproject.toml 来确定需要启动的虚拟环境
poetry shell

# 查看虚拟环境信息，默认查看当前虚拟环境信息
poetry env info 
# -p 参数指定虚拟环境的路径，可以查看具体的虚拟环境信息

# 删除指定的虚拟环境
poetry env remove [Python 版本或虚拟环境路径]

~~~

### 虚拟环境依赖包管理
~~~shell
# 修改 poetry 镜像源为清华镜像源
poetry source add tsinghua https://pypi.tuna.tsinghua.edu.cn/simple


# 根据 pyproject.toml 配置文件安装依赖
poetry install 
# --no-cache 命令可以禁用缓存，并强制从远程源（如 PyPI）重新下载所有依赖项，而不是使用本地缓存中的包
# 如果没有虚拟环境，poetry install 会自动创建一个虚拟环境并安装依赖项。如果已经有虚拟环境，poetry install 会在现有的虚拟环境中安装依赖项

# 在虚拟环境中安装依赖（类似于在虚拟环境在pip install），同时 pyproject.toml 配置文件会发生变化
poetry add [包名]
# 当使用 poetry add 指令时，poetry 会自动依序帮你做完这三件事：
# 1. 更新 pyproject.toml
# 2. 依照 pyproject.toml 的内容，更新 poetry.lock，它实际上就相当于 pip 的 requirements.txt ，详细记录了所有安装的模块与版本
# 3. 依照 poetry.lock 的内容，更新虚拟环境
# poetry.lock 的内容是取决于 pyproject.toml ，但两者并不会自己连动，一定要基于特定指令才会进行同步与更新， poetry add 就是一个典型案例。

# 更新 poetry.lock
poetry lock
# 当你自行修改了 pyproject.toml 内容，比如变更特定模块的版本（这是有可能的，尤其在手动处理版本冲突的时候）
# 此时 poetry.lock 的内容与 pyproject.toml 出现了脱钩，必须要执行 poetry lock 使poetry.lock依照新的 pyproject.toml 内容更新、同步
# 注意的是，poetry lock 指令，仅会更新 poetry.lock ，不会同时安装模块至虚拟环境
# 因此，在执行完 poetry lock 指令后，必须再使用 poetry install 来安装模块。否则就会出现 poetry.lock 和虚拟环境不一致的状况。

# 显示虚拟环境的所有依赖项信息（类似 pip list）
poetry show [指定包名，显示特定包的详细信息]
# --tree 以树状结构显示依赖项及其子依赖项的详细信息
# -v 显示所有已安装包的详细信息


# 移除虚拟环境的模块，pip 的 pip uninstall 没有依赖解析功能，只会移除你所指定的模块，而不会连同依赖模块一起移除
poetry remove [包名]

~~~

