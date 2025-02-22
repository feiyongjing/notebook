# uv简介
uv 是一个非常快速的 Python 依赖安装程序和分解器，使用 Rust 编写，旨在替代 pip 和pip-tools 工作流，速度比他们快 8～10 倍，当前可用于替代 pip, pip-tools, virtualenv等

## 安装 uv 工具（支持多种方式安装）
### pip安装
~~~shell
# pip
pip install uv
~~~

### macOS和linux安装 
~~~shell
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh
~~~

### windows安装
~~~shell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
~~~

## uv工具主要使用的两个文件
- pyproject.toml：定义项目的主要依赖，包括项目名称、版本、描述、支持的 Python 版本等信息
- uv.lock：记录项目的所有依赖，包括依赖的依赖，且跨平台，确保在不同环境下安装的一致性。这个文件由 uv 自动管理，不要手动编辑

## uv项目管理
~~~shell
# uv安装python
uv python install 3.12

# 查看uv安装的python列表
uv python list

# 创建项目，uv init 创建的项目会自动使用git进行管理
uv init [project-name] [-r requirements.txt]

# 根据pyproject.toml进行安装下载依赖，同时修改pyproject.toml新增和移除依赖之后也是一样同步依赖库
# 并且创建一个项目默认的虚拟环境，默认的虚拟环境在当前目录下的 .venv 目录中
uv sync

# 在虚拟环境之外直接执行uv run 会使用项目默认的虚拟环境
uv run
# 无需指定程序入口，一般是配置在pyproject.toml中或者其他框架的配置文件中
# 当然在虚拟环境之外也可以通过 --python 指定一个虚拟环境的python解释器（虚拟环境）执行，例如 uv run --python .venv/bin/python3.12 -- test.py
~~~

## 虚拟环境管理
~~~shell
# 创建虚拟环境，创建指定python版本的虚拟环境，创建完成后当前目录下会出现一个和虚拟环境name相同的目录
uv venv --python [python可执行程序完整路径] [虚拟环境name]

# 激活虚拟环境
source [虚拟环境name/bin/activate]

# 退出虚拟环境
deactivate

# 在虚拟环境之外直接执行uv run 会默认在当前目录创建一个虚拟环境，默认的虚拟环境在当前目录下的 .venv 目录中
uv run [test.py]
# 如果是没有依赖的脚本执行请添加 --no-project 例如 uv run --no-project test.py

# 默认虚拟环境安装依赖，执行下面的命令或者编辑pyproject.toml
uv pip install [package-name]
uv add [package-name]

# 默认虚拟环境卸载依赖，执行下面的命令或者编辑pyproject.toml
uv pip uninstall [package-name]
uv remove [package-name]

# 默认虚拟环境更新依赖，--upgrade-package 标志将尝试将指定的包更新到最新的兼容版本，同时保持锁文件的其余部分不变
uv pip install --upgrade [package-name]
uv lock --upgrade-package requests

# 默认虚拟环境根据依赖清单批量安装依赖
uv pip install -r requirements.txt

# 默认虚拟环境生成依赖清单
uv pip freeze > requirements-3.in

# 默认虚拟环境通过 requirements.in 生成 requirements.txt 
uv pip compile requirements.in -o requirements.txt 

# 默认虚拟环境通过 pyproject.toml 生成 requirements.txt 
uv pip compile pyproject.toml -o requirements.txt 
~~~




