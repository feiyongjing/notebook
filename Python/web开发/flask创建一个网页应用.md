# 搭建一个web应用

### 安装 Flask 框架
~~~
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple flask
~~~
---

### 使用 Flask 搭建web应用
~~~python
# 导入 Flask 模块 和 request模块
from flask import Flask, request

# 并创建一个应用对象
app = Flask(__name__)


# 设置路由
@app.route("/", methods=["GET"])
def index():
    return "Hello, World!"


@app.route("/login", methods=["POST"])
def login():
    username = request.json["username"]
    password = request.json["password"]

    # request.args["username"] 获取url后面的param解析成字典
    # request.form["username"] 获取body中的数据当成表单（application/x-www-form-urlencoded）解析成字典
    # request.json["username"] 获取body中的数据当成json解析成字典

    if username != "admin" and password != "admin":
        return {"code": "999999", "message": "用户名密码错误"}
    return {"code": "000000", "message": "登录成功"}


# 路径参数的获取
@app.route("/api/login/<username>/<password>", methods=["POST"])
def apiLogin(username, password):
    if username != "admin" and password != "admin":
        return {"code": "999999", "message": "用户名密码错误"}
    return {"code": "000000", "message": "登录成功"}

# 启动应用程序。可以使用 Flask 的 run 函数来启动应用程序
if __name__ == '__main__':
    app.run()

~~~
---
