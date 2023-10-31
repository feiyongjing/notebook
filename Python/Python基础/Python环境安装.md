### 参考官网
https://www.python.org/

# python是解释型语言，即python解释器负责将python源码逐行翻译成机器码给cpu执行（一边翻译一般执行，执行效率没有编译型语言高）
### 常见的python解释器
1. python 和 python3 分别是python2和python3的解释器
2. CPython是官方使用C语言编写的python解释器，Pycharm一般会将模块的通过CPython编译成 .pyc 文件，本质是一个字节码的二进制文件可以直接运行，这样就无需解释器重复的接收模块代码加快运行速度
3. Jython是java语言编写的python解释器，可以运行在java平台
4. IronPython可以运行在 .NET 和 Mono 平台
5. PyPy是Python语言编写的解释器，支持JIT即时编译


# Python代码的运行方式

## 方式一：直接使用解释器运行 .py 文件
~~~shell
python test.py
~~~
---

## 方式二：Linux下使用脚本方式运行 .py 文件
~~~shell
# 第一步：给需要运行的 .py 文件开头添加 shebang符号并且后面接python解释器程序的路径 (通过which python去查询解释器的路径 ，一般情况下都是 /usr/bin/python 这个路径)，即 #! python解释器程序的路径，其实和bash脚本开头的声明是一样的
# 第二步：给需要运行的 .py 文件添加可执行权限
# 第三步：使用绝对路径或相等路径直接运行脚本
./test.py
~~~
---

## 方式三：使用解释器进入交互式终端直接编写代码执行
~~~shell
# 进入终端
python 

# 编写代码直接运行
print("哈哈")

# 退出交互式终端 或者使用 Ctrl键 + d键 退出
exit()
~~~
---

## 方式四：使用ipython进入交互式终端直接编写代码执行，并且交互式终端还支持linux终端命令，编写代码和输入linux命令都支持tab键补全，但是注意python的最新版本可能不支持ipython
~~~shell
# 安装ipython，如果安装慢需要使用清华源
pip install ipython

# 进入ipython终端
ipython

# 编写代码直接运行，注意如果是常量无需使用print打印，直接输入常量回车就会打印常量，如果是常量赋值给变量可以直接输入变量名打印数据或者使用 print(变量名)
a="哈哈"
print(a)

# 输入linux命令也可以运行
ls -alth

# 退出交互式终端 直接使用linux的 exit 也是可以的
exit()
~~~
---

## 方式五：Pytharm运行代码

## 方式六：Jupyter Notebook运行代码

### pip安装运行Jupyther
~~~shell
# 注意jupyter都不支持最新版本的python，jupyter的大版本需要与python的版本对应
# 推荐python版本 3.9 这里使用的是3.9.10
pip install jupyter

# 启动Jupyter
jupyter notebook --port [port_number]
# --port参数指定启动端口，不加默认是8888端口，并且端口被占用会自动向上加一端口号
# --no-browser参数不会在服务启动后自动打开浏览器
# --debug 设置debug模式启动，可以看到debug日志信息

# 内核连接不上请 pip uninstall pyzmq 再 pip install pyzmq==19.0.2

# Jupyter 设置内核添加、修改、删除参考
# https://blog.csdn.net/I_LOVE_MCU/article/details/108311698#:~:text=%E4%B8%BA%20Jupyter%20notebook%20%E6%B7%BB%E5%8A%A0%E5%86%85%E6%A0%B8%20python%20-m%20ipykernel,install%20--user%20--name%3Dkernelname%20--display-name%20showname%201%20%E5%85%B6%E4%B8%AD%EF%BC%8Ckernelname%E4%B8%BA%E5%88%9B%E5%BB%BA%E7%9A%84%E6%96%87%E4%BB%B6%E5%A4%B9%E5%90%8D%EF%BC%8Cshowname%E4%B8%BA%E5%9C%A8Jupyter%20notebook%E5%B1%95%E7%A4%BA%E7%9A%84%E5%86%85%E6%A0%B8%E5%90%8D

# Jupyter 查看已有的内核名称和内核配置文件目录，在目录下有 kernel.json 可以配置该内核的解释器
jupyter kernelspec list

# 查看是否安装内核
python -m ipykernel --version

# 安装ipykernel内核
python -m pip install ipykernel

# 为 Jupyter notebook 添加内核
python -m ipykernel install --user --name=kernelname  --display-name showname
# kernelname 为创建的文件夹名
# showname 为在Jupyter notebook展示的内核名

# 删除 Jupyter notebook 设置的内核
jupyter kernelspec remove kernelname
# kernelname 为创建的文件夹名

~~~
---

### Anaconda安装运行Jupyther
~~~shell

# 创建环境
conda create --name jupyter-test jupyter python=3.9

# 进入环境
conda activate jupyter-test

# 菜单汉化参考：https://zhuanlan.zhihu.com/p/549843517
# 添加用户环境变量 LANG  值是 zh_CN.UTF8

# 去 https://anaconda.org 搜索需要的包和版本
# 安装 19.0.2 版本的 pyzmq
conda install -c jasonb857 pyzmq

# 启动Jupyter
jupyter notebook --port [port_number]
# --port参数指定启动端口，不加默认是8888端口，并且端口被占用会自动向上加一端口号
# --no-browser参数不会在服务启动后自动打开浏览器
~~~
---

