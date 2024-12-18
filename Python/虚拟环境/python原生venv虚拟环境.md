参考：https://liaoxuefeng.com/books/python/built-in-modules/venv/


### 第一步，创建目录，这里把Python虚拟运行环境命名为proj101env，因此目录名为proj101env
~~~shell
mkdir proj101env
cd proj101env/
~~~


### 第二步，创建一个独立的Python运行环境
~~~shell
# 创建一个独立的Python运行环境
python3 -m venv .


# 查看当前目录，可以发现有几个文件夹和一个pyvenv.cfg文件：
ls
# bin  include  lib  pyvenv.cfg
# 观察bin目录的内容，里面有python3、pip3等可执行文件，实际上是链接到Python系统目录的软链接。

# 继续进入bin目录，Linux/Mac用source activate，Windows用activate.bat激活该venv环境
cd bin
# source activate 激活该venv环境，注意到命令提示符变了，有个(proj101env)前缀，表示当前环境是一个名为proj101env的Python环境
# 下面就正常安装各种第三方包，并运行python命令

# 在venv环境下，用pip安装的包都被安装到proj101env这个环境下，具体目录是proj101env/lib/python3.x/site-packages，因此，系统Python环境不受任何影响。也就是说，proj101env环境是专门针对project101这个应用创建的

# deactivate命令退出当前的proj101env环境
deactivate

# 如果不再使用某个venv，例如proj101env，删除它也很简单。首先确认该venv没有处于“激活”状态，然后直接把整个目录proj101env删掉就行
~~~




