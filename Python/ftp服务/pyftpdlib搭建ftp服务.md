## pyftpdlib快速搭建ftp服务
~~~shell
# 安装 pyftpdlib
pip install pyftpdlib

# 简单共享：安装完成后，找到需要共享的目录，然后启动命令行运行命令，之后浏览器登陆ftp://ip:port,这样就开启了一个最简单的ftp共享服务。
python -m pyftpdlib -p [port]

~~~

## 搭建ftp服务并且设置用户权限
简单的ftp搭建方式，肯定不满足我们的要求，那么就需要进行二次开发了！但也仅仅需要几行代码而已：
~~~python
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer
from pyftpdlib.authorizers import DummyAuthorizer

authorizer = DummyAuthorizer()
authorizer.add_user('python', '123456', 'F:\\Working~Study', perm='elradfmwM')
handler = FTPHandler
handler.authorizer = authorizer

server = FTPServer(('0.0.0.0', 8888), handler)
server.serve_forever()
~~~