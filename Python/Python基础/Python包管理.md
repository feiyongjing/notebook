# 安装使用第三方的包
### pip命令安装第三方包
### 清华源参考：https://mirrors.tuna.tsinghua.edu.cn/help/pypi/

### pip 是 Python 2.x 版本的包管理器，而 pip3 是 Python 3.x 版本的包管理器
### 如果您只安装了 Python 3.x，那么 pip 和 pip3 实际上是相同的，因为它们都是 Python 3.x 版本的包管理器
~~~shell

# pip 安装 第三方包 只写报名就安装最新的版本，如果速度比较慢请使用 清华源
pip install [包名==版本]

# 查看已经安装的包
pip list

# pip 卸载包
pip uninstall [包名]

# 临时使用 清华源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple [包名==版本]

# 升级 pip 到最新的版本 (>=10.0.0) 后进行设置长期默认使用 清华源
python -m pip install --upgrade pip
#  pip 默认源的网络连接较差，临时使用清华源来升级 pip
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
# 设置长期默认使用 清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 导出全部包
pip freeze > [requirements.txt]

# 卸载指定清单的包列表
pip uninstall -r [requirements.txt] -y

# 查看已经安装的包版本
pip show [包名]
# 其他查看方式如下
# pip list | grep [package-name]
# pip freeze | grep [package-name]

~~~
---

# 自定义模块打包给其他人使用

### 第一步：在自定义模块目录下创建一个名为 setup.py 的文件，内容如下
~~~python
from setuptools import setup

setup(
    name='my_package',   # name 参数指定包的名称
    version='0.1',       # version 参数指定包的版本号
    description='My custom Python package',           # description 参数指定包的描述信息
    author='Your Name',                               # author 参数指定包的作者
    author_email='your_email@example.com',            # author_email 参数指定作者的电子邮件地址
    packages=['my_module', 'my_module.test'],         # packages 参数指定包含的模块或子包的名称
    install_requires=[                                # install_requires 参数指定包的依赖项
        'numpy>=1.21.2',
        'pandas>=1.3.3',
    ],
)
~~~
---

### 第二步：包目录下打开命令行终端，执行以下命令来生成打包后的 Python 包
### 该命令会在包目录下生成一个名为 dist 的目录，其中包含了打包后的 Python 包。打包后的 Python 包的文件名格式为 <包名>-<版本号>.tar.gz，例如 my_package-0.1.tar.gz
~~~shell
python setup.py sdist
~~~
---

### 第三步：将打包后的 Python 包分享给其他人。其他人可以通过以下命令来安装该包
~~~shell
pip install <包名>-<版本号>.tar.gz
# 例如 pip install my_package-0.1.tar.gz
~~~
---

### 第四步：安装完成后，就可以在 Python 中导入该包的模块
~~~python
import my_module
~~~
---

### 第五步：卸载该包
~~~shell
pip uninstall my_package
~~~
---