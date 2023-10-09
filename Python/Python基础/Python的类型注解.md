# 类型注解可以使编译器识别代码提高开发效率
# 注意类型注解的本质只是便于观看程序，并不保证类型一定正确，错误的类型注解编译和运行都不会报错

# 变量赋值的类型注解
~~~python
# 类型注解，类型注解使程序的可读性提高
var_1: int = 123
var_2: str = "habx"
var_3: bool = True
var_4: list = [1, 2, 3]
var_5: tuple = (1, "asw", True)
var_6: set = {1, 2, 3}
var_7: dict = {"id": 1, "name": "张三", "age": 3}

# 复杂类型的注解(其实就是泛型)
var_8: list[int] = [1, 2, 3]
var_9: tuple[int, str, bool] = (1, "asw", True)
var_10: set[int] = {1, 2, 3}
var_11: dict[str, int] = {"id": 1, "name": 555, "age": 3}

# 类型注解的其他写法，在代码的末尾添加注释 type: 类型
var_12 = "axw"  # type: str

# 以下错误的类型注解编译和运行都不会报错
var_13: int = "123"
var_14: str = 123
~~~
---

### 函数的类型注解：设置参数和返回值的类型，同样不保证类型一定正确，错误的类型注解编译和运行都不会报错
~~~python
# 模板如下
# def 函数名(参数名称1: 参数1的类型, 参数名称2: 参数2的类型, ...) -> 返回值的类型:
#     pass

def fn(age: int, name: str) -> str:
    return f"名称是：{name}，年龄是：{age}"

print(fn(17, "奈德丽"))
~~~
---


### 联合类型Union：Union类型表示具体的类中在那几种类型之中
~~~python
# 需要导入内置模块
from typing import Union

# 使用Union对类型不统一的集合变量或其他类型变量进行类型注解
var_1: list[Union[int, str]] = [1, "tgxcs", 3]
var_2: set[Union[int, str]] = {1, "andos", 3}
var_3: dict[str, Union[int, str]] = {"id": 1, "name": "张三", "age": 3}

# 使用Union对函数的参数和返回值设置类型注解
def fn(age: int, name: str) -> Union[str, int]:
    if age > 18:
        return f"名称是：{name}，年龄是：18"
    else:
        return age


print(fn(20, "奈德丽"))
~~~
---