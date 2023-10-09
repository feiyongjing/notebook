### 简便版本（单线程阻塞）
~~~python
import socket

# 创建一个 TCP/IP socket
server_socket = socket.socket()

# 绑定 socket 地址和端口号
server_address = ('localhost', 8888)
server_socket.bind(server_address)

# 监听 socket 地址的端口号
server_socket.listen(1)

# 等待客户端连接
print('等待客户端连接...')
# accept()函数阻塞等待连接，连接成功返回一个元组
# 0号索引元素是客户端与服务端的链接对象，1号索引元素是客户端地址信息
conn, client_address = server_socket.accept()
print(f'客户端 {client_address} 连接成功！')

while True:
    # recv()函数会阻塞等待客户端发送的消息，参数是接收数据的缓冲区大小，单位是字节
    data: str = conn.recv(1024).decode("UTF-8")
    print("客户端发来的消息是：", data)

    # 向客户端发送欢迎消息
    message = input("请输入回复客户端的信息：")
    if message == "exit":
        conn.sendall("服务端主动断开了连接".encode("UTF-8"))
        # 关闭连接
        conn.close()
        break
    else:
        conn.sendall(message.encode("UTF-8"))

# 关闭服务端
server_socket.close()

~~~
---

# 完整版
### 可以使用 netassist 网络调试工具连接调试，或者是socket客户端进行连接
~~~python
import socket
from concurrent.futures import ThreadPoolExecutor
import collections
import threading

# 线程安全的字典类
class ThreadSafeDict:
    def __init__(self):
        self._dict = collections.defaultdict(lambda: None)
        self._lock = threading.Lock()

    def __getitem__(self, key):
        with self._lock:
            return self._dict[key]

    def __setitem__(self, key, value):
        with self._lock:
            self._dict[key] = value

    def __delitem__(self, key):
        with self._lock:
            del self._dict[key]

    def __len__(self):
        with self._lock:
            return len(self._dict)

    def __contains__(self, key):
        with self._lock:
            return key in self._dict

    def clear(self):
        with self._lock:
            self._dict.clear()

    def keys(self):
        with self._lock:
            return list(self._dict.keys())

    def values(self):
        with self._lock:
            return list(self._dict.values())

    def items(self):
        with self._lock:
            return list(self._dict.items())


def task(server_socket, tsDict: ThreadSafeDict):
    # accept()函数阻塞等待连接，连接成功返回一个元组
    # 0号索引元素是客户端与服务端的链接对象，1号索引元素是客户端地址信息
    conn, client_address = server_socket.accept()
    print(f'客户端 {client_address} 连接成功！')

    name = registerUsername(conn)

    while True:
        singleLinkMessage(conn, f"聊天室的成员列表：{tsDict.keys()}")
        singleLinkMessage(conn, f"当前自己的昵称是：{name}")
        singleLinkMessage(conn, f"请输入需要发送给谁消息，格式是 昵称:消息")

        # 客户输入 用户名:消息 给指定的用户发送消息
        # 客户输入 exit 退出聊天室
        # 客户输入 @all:消息 给聊天室全部用户发送消息
        data: str = conn.recv(1024).decode("UTF-8")
        print(f"客户端 {client_address} {name}发来的消息是：", data)

        # 当前用户是否主动断开连接或者是手动退出聊天室通知所有人
        if not data or data.strip() == 'exit':
            broadcastMessage(tsDict.values(), f"{name}下线了")
            # 关闭连接
            conn.close()
            tsDict.__delitem__(name)
            break

        if ":" not in data:
            # 输入错误给当前用户的提示
            singleLinkMessage(conn, "消息格式错误，请重新输入")
            continue

        username,message = data.split(":", 1)
        if username=='@all':
            # 发送消息给聊天室的全部人
            broadcastMessage(tsDict.values(), f"{name}@所有人:{message}")
        elif tsDict.__contains__(username):
            # 发送消息给聊天室的指定用户
            singleLinkMessage(tsDict.__getitem__(username), f"{name}@你:{message}")
        else:
            # 输入错误给当前用户的提示
            singleLinkMessage(conn, "输入的用户名称不在聊天室中，请重新输入")

def registerUsername(conn) -> str:
    name: str
    singleLinkMessage(conn, "请输入用户名：")
    while True:
        name = conn.recv(1024).decode("UTF-8").strip()
        if name == '@all' or ':' in name:
            singleLinkMessage(conn, "不支持的用户名，请重新输入用户名")
        elif tsDict.__contains__(name):
            singleLinkMessage(conn, "该用户名已经被聊天室的其他人占用了，请重新输入用户名")
        else:
            tsDict.__setitem__(name, conn)
            broadcastMessage(tsDict.values(), f"{name}上线了")
            break
    return name


# 单发消息给用户
def singleLinkMessage(clientConn, message: str):
    clientConn.sendall(message.encode("UTF-8"))

# 广播消息给全部用户
def broadcastMessage(conns, message: str):
    for conn in conns:
        singleLinkMessage(conn, message)

if __name__ == '__main__':
    # 创建一个 TCP/IP socket
    server_socket = socket.socket()

    # 绑定 socket 地址和端口号
    server_address = ('localhost', 8888)
    server_socket.bind(server_address)

    # 监听 socket 地址的端口号
    server_socket.listen(1)

    # 等待客户端连接
    print('等待客户端连接...')

    # 线程安全的字典存放用户名和对应的客户端连接
    tsDict = ThreadSafeDict()

    # 开启50个线程，每个线程都等待一个客户端连接
    with ThreadPoolExecutor(50) as t:
        while True:
            t.submit(task, server_socket, tsDict)

    # 关闭服务端
    server_socket.close()

~~~
---