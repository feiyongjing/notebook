### 字符串的操作一般不会对原字符串产生影响，一般都是返回一个新的字符串

### 字符串的相加、字符串与数字的相乘、列表拼接为字符串
~~~python
# 字符串使用双引号或者是单引号表示，当然3个单引号或者是双引号也表示字符串并且可以表示多行字符串
str0='abc'

# 字符串可以与字符串相加，字符串可以与数字相乘(表示字符串重复的次数)，但是字符串无法与数字相加
str1='abc'+'123'
print(str1)
str2=str1 * 3
print(str2)

# input函数会将参数打印到屏幕上，然后等待输入，在输入完按下回车会将输入的内容转换为字符串返回
print('加法计算器')
a=input('请输入第一个数字')
print(type(a)) # type()函数检查变量的类型
b=input('请输入第二个数字')
print('输出的结果是')
print(int(a)+int(b))

# join() 函数将列表list拼接为字符串
print("_".join(["Java","JavaScript","C","C++","Python"]))

~~~
---

### 字符串拼接的占位符
~~~python
# 字符串拼接的占位符,推荐使用最后一种 f-string 的写法

# %s 表示字符串
# %d 表示整数
# %f 表示float
# %% 表示输出%

name="胡图图"
address="翻斗花园3栋201"
age=5
hobby="丢沙包、骑自行车"

s0 = "我叫%s，我家住在%s，我今年%d岁了，我的爱好是%s" % (name, address, age, hobby)
s1 = "我叫{}，我家住在{}，我今年{}岁了，我的爱好是{}".format(name, address, age, hobby)
s2 = f"我叫{name}，我家住在{address}，我今年{age}岁了，我的爱好是{hobby}"  # 字符串前面添加 f 表示字符串是模板字符串

print(s0)
print(s1)
print(s2)

print(r"E:\workspace\notebook")  # 字符串前面添加 r 表示是一个原始字符串，即字符串中的转义字符（如 \n、\t 等）不会被转义，通常用于正则表示式或文件的路径
~~~
---


### 字符串的索引和截取
~~~python
s="你好啊，我叫赛丽亚"

# 通过索引获取字符串中的字符，如果是负数就是从后向前获取字符，但是无法超过边界否则抛出异常
print(s[5])

# 字符串的截取
# s[start:end:step]
# step表示步长，默认不写是1，正数代表从左向右截取字符串，负数表示从右向左截取字符串，并且数字表示每几个字符中取开头的字符
# step如果是正数那么start的索引必须在end索引左边，step如果是负数数那么start的索引必须在end索引右边
# 截取的字符串包含start索引位置的字符但是不包含end索引位置字符
# 如果start省略就是截取字符串开头到end索引位置字符的字符串
# 如果end省略就是截取start索引位置到字符串的结尾的子字符串
# 如果是 s[:] 就还是原来的字符串
print(s[2:5])

s1="xqg7bd810gh7e021tye021e7y"
print(s1[-2:5:-3])

~~~
---

### 字符串的替换、去除首尾空格、分割
~~~python
# replace()函数进行字符串替换
s="你好啊，我叫张三"
print(s.replace("张三", "赛丽亚"))

# strip()函数进行字符串首尾空白字符的去除
username="   赛丽亚  "
password=" 123456  "
print("用户名是"+username.strip())
print("密码是"+password.strip())

# split()函数分割，按照参数分割字符串返回list列表
a="Java_JavaScript_C_C++_Python"
lst=a.split("_")
print("目前学习的编程语言有", lst)
~~~

### 字符串的查找和判断
~~~python
# find()函数和index()函数查找子字符串的开始索引，可以根据这个判断是否包含子字符串
# 注意find()函数在子字符串不存在时返回-1，而index()函数在子字符串不存在时抛出异常
s="你好啊，我叫赛丽亚"
print("字符串是否包含子字符串", s.find("赛丽亚")!=-1)
print("字符串是否包含子字符串", s.index("赛丽亚")>=0)

# in 和 not in 关键字也可以用于判断字符串是否包含子字符串
print("in 关键字判断字符串是否包含子字符串",  "赛丽亚" in s )
print("not in 关键字判断字符串是否不包含子字符串",  "赛丽亚" not in s )

# startswith函数和endswith函数判断字符串是否包含指定前缀和后缀
print("判断字符串是否包含指定前缀",  s.startswith("你好吗") )
print("判断字符串是否包含指定后缀",  s.endswith("赛丽亚") )

# isdigit函数判断字符串是否整数组成
print("判断字符串是否整数组成",  "123".isdigit())
~~~


