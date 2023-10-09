### 一个简单的客户端
~~~python
import socket
from threading import Thread


def read(server_socket):
    while True:
        data: str=server_socket.recv(1024)
        # 判断套接字是否已经关闭
        if not data:
            break
        print(data.decode("UTF-8"))


def write(server_socket):
    while True:
        message: str = input("")
        server_socket.sendall(message.encode("UTF-8"))
        if message.strip()=='exit':
            break



if __name__ == '__main__':
    # 创建一个 TCP/IP socket
    server_socket = socket.socket()

    # 客户端连接服务端的地址和端口号
    server_address = ('localhost', 8888)
    server_socket.connect(server_address)

    # 开启两个线程，一个监听服务的返回的数据并打印，另一个监听用户的输入将输入发送给服务器
    t = Thread(target=read, args=(server_socket,))
    t1 = Thread(target=write, args=(server_socket,))
    t.start()
    t1.start()

    # join()方法会阻塞主线程，直到被调用的线程执行完毕或者超时
    t.join()
    t1.join()


# 关闭客户端
server_socket.close()

~~~
---