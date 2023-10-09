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

# 安装使用第三方的包
### pip命令安装第三方包
### 清华源参考：https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
~~~shell

# pip 安装 第三方包 如果速度比较慢请使用 清华源
pip install [包名]

# 临时使用 清华源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple [包名]

# 升级 pip 到最新的版本 (>=10.0.0) 后进行设置长期默认使用 清华源
python -m pip install --upgrade pip
#  pip 默认源的网络连接较差，临时使用清华源来升级 pip
python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
# 设置长期默认使用 清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

~~~
---