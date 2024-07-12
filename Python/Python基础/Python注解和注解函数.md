# 注解和注解函数
- 注解是标注在函数上的一个标记，当执行这个函数时会有切入点（函数执行前或执行后等时机），在切入点执行标记注解对应的注解函数
- 注解函数的名称就是注解名称

# 常见的注解
- @staticmethod 标注的方法是静态方法，可以通过类或实例调用，但是它们不会自动传递类（cls）或实例（self）作为第一个参数
- @classmethod 标注的方法是类，方法可以通过类或实例调用，但是它们会自动传递类（cls）作为第一个参数
- 

### 自定义注解装饰器案例
~~~python
# 通过wrapper装饰器增强目标函数并将增强的函数返回，由调用者决定何时调用增强的函数
def wrapper(fn):
    # inner函数声明的 *args, **kwargs 形式参数是被增强函数的原有参数的打包
    def inner(*args, **kwargs):
        print("在目标之前做一些事情")
        # 调用函数时的 *, ** 表示将 args列表和 kwargs字典打散成位置参数和关键字参数
        res = fn(*args, **kwargs)
        print("在目标之后做一些事情")
        return res

    return inner


# 添加注解增强函数，注解名称就是装饰器函数
@wrapper
def play_dnf(username, password):
    if (username == "admin" and password == "123456"):
        print("今天也是充满7万的一天")
        return "深渊爆出了史诗装备氤氲之息"
    else:
        print("登录失败")
        return "请重新输入用户名密码"


# 直接调用函数
print(play_dnf("admin", "123456"))
~~~
---