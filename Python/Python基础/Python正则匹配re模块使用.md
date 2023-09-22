### 提取到正则匹配的完整字符串
~~~python
import re

# 正则表达式前面的r表示字符串是原始字符串，内部的转义字符不生效

# findall() 函数返回一个列表，存放正则表达式匹配的字符串
lst = re.findall(r"\d+", "我的电话号码是：10086，我女朋友的电话是10010")
print(lst)

# finditer() 函数返回一个迭代器，需要通过迭代元素match对象调用group()获取正则表达式匹配的字符串
iter = re.finditer(r"\d+", "我的电话号码是：10086，我女朋友的电话是10010")
for i in iter:
    print(i.group())

# search() 函数中不会返回正则表达式匹配全部匹配的结果，而是匹配到一个字符串就直接返回一个match对象，match对象调用group()获取正则表达式匹配的字符串
m = re.search(r"\d+", "我的电话号码是：10086，我女朋友的电话是10010")
print(m.group())

# match() 函数中默认给正则表达式最前面添加了^ 只匹配字符串的开头，返回一个match对象，match对象调用group()获取正则表达式匹配的字符串
m = re.match(r"\d+", "10086，我女朋友的电话是10010")
print(m.group())

# 正则预编译：提前编译正则表达式可以优化程序的运行速度
obj = re.compile(r"\d+")
# 预编译的正则对象可以正常使用上述的findall()、finditer()、search()、match()函数
print(obj.findall("我的电话号码是：10086，我女朋友的电话是10010"))

~~~
---

### 单独提取到正则匹配的字符串
~~~python
import re

# (?P<分组名称>正则) 可以单独提取到正则匹配的字符串
obj = re.compile(r"<div id=(?P<id>\d+)><spen>(?P<location>.*?)</spen>(?P<name>.*?)</div>")

s = "<div id=1><spen>上单</spen>放逐之刃 锐雯</div>" + \
    "<div id=2><spen>中单</spen>九尾妖狐 阿狸</div>" + \
    "<div id=3><spen>打野</spen>傲之追猎者 雷恩加尔</div>"

iter = obj.finditer(s)

# 调用group()函数通过分组名称获取单独提取到正则匹配的字符串
for i in iter:
    print(i.group("id"), i.group("location"), i.group("name"))

~~~
---