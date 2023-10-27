### 模块的导入方式
~~~python
# 导入语法，只使用import会导入模块的全部功能，from加import只会导入模块的一部分功能，模块的导入写到文件的开头部分
# * 表示导入全部的模块并且使用时模块名称可以简写省略，例如 from time import * 后 可以直接使用time()函数，而无需time.time{}
[from 模块名] import [模块 | 类 | 变量 | 函数 | *] [as 别名]

# 常见的组合形式
import [模块名]

from [模块名] import [类 | 变量 | 函数 | *]

import [模块名] as [别名]

from [模块名] import [类 | 变量 | 函数 | *] [as 别名]

# 自定义模块的名称就是文件的名称
[from 自定义模块文件的名称] import [模块 | 类 | 变量 | 函数 | *] [as 别名]

# 导入同一名称的 模块、类、变量、函数 情况下，后导入的模块、类、变量、函数 会覆盖先导入，只有后导入的生效
~~~
---


### __name__ 内置变量
~~~python

# 注意当导入一个模块时该模块中没有缩进的代码块就会全部执行一遍
# 而为了防止执行不必要的代码（这段代码可能会需要）执行一般都是使用  if __name__ == '__main__' 判断代码块是否执行

# __name__是一个内置变量，当以当前文件作为启动文件时该变量就是 '__main__' 就会执行代码块
# 而当前文件作为自定义模块导入时，以其他文件作为启动文件时该变量不是 '__main__' 就不会执行代码块
if __name__ == '__main__':
    print(3)

~~~
---


### __all__变量的使用：进行封装只暴露必要的函数，不必要的函数不暴露
如果导入的模块文件并且导入文件是导入的 * ，并且模块文件有声明变量 __all__ 则只能使用 __all__ 变量中的函数
当然如果不是导入的 * 而是手动导入函数是没有 __all__ 变量的限制的
~~~python
__all__=['fn1']

def fn1():
    print(1)


def fn2():
    print(2)
~~~
---

# 包
### 包由 __init__.py 文件和模块文件组成，并且包中必须有 __init__.py 文件，在pycharm中创建的包会自动创建空的 __init__.py 文件

### 包中模块引用如下，并且引入 * 也会收到 __all__变量 的限制，但是注意 __all__ 变量是在 __init__.py 文件声明赋值的
~~~python

from [包名] import [模块名 | 类 | 变量 | 函数 | *] [as 别名]

from [包名.模块名] import [类 | 变量 | 函数 | *] [as 别名]

~~~
---

# 模块的 __file__ 变量
~~~python

import threading

# 模块的 __file__ 变量会打印模块的所在目录（包括系统模块或者是自定义模块所在的目录）
print(threading.__file__)
~~~
---

# 安装使用第三方的包
### pip命令安装第三方包
### 清华源参考：https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
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