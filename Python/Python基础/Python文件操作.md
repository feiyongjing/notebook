### 读取文件
~~~python
# mode选择模式
# 'r'读取(默认)
# 'w'打开写入，先截断文件
# 'x'创建一个新文件并打开它进行写入
# 'a'打开用于写入，如果存在，则追加到文件末尾
# 'b'二进制模式
# 't'文本模式(默认)
# '+'打开磁盘文件进行更新(读写)

# mode="rb" rb读取非文本的二进制文件，并且无需添加编码方式 f=open("高能图片.webp", mode="rb")
f=open("常见英雄.json", mode="r", encoding="utf-8")

# 读取全部内容，注意文件过大是无法读取的，同时指针移动到文件末尾
# content= f.read()
# print(content)

# 读取一行内容，注意读取出来的内容包含换号符，同时指针移动到下一行
# print(f.readline())
# print(f.readline())

# 读取全部内容，返回一个列表，列表每一个元素就是文件的一行，注意元素中包含换号符
# content = f.readlines()
# print(content)

# 一行行的读取文件，并且去除每行的首尾空白字符（包含空格、换行\n、回车\r、制表符\t）
for line in f:
    print(line.strip())

# 注意关闭文件
f.close()


# with关键字方式进行打开文件，离开作用域会自动的关闭文件，无需手动关闭
with open("常见英雄.json", mode="r", encoding="utf-8") as f:
    for line in f:
        print(line.strip())
~~~
---


### 写文件
~~~python
# mode="w" 以编辑模式打开文件，文件不存在会创建文件，如果文件存在会清空文件内容
# mode="a" 以追加模式(append)打开文件
f = open("不常见英雄.json", mode="w", encoding="utf-8")

lst = ['{', '"恶魔小丑": "萨科",', '"琴瑟仙女": "娑娜",', '"迅捷斥候": "提莫"', '}']

for item in lst:
    f.write(item)
    f.write("\n")

# 注意关闭文件
f.close()


# with关键字方式进行打开文件，离开作用域会自动的关闭文件，无需手动关闭
with open("不常见英雄.json", mode="w", encoding="utf-8") as f:
    for item in lst:
        f.write(item)
        f.write("\n")

~~~
---

### 同时读写文件
~~~python
# mode="r+" 表示同时读写文件
with open('example.txt', mode="r+", encoding="utf-8") as file:
    content = file.read()  # 读取原文件内容
    print(content)         # 输出原文件内容
    file.write('Nbx8o3fg46o23')  # 向文件中写入新内容
    file.seek(0)                 # 将文件指针移动到文件开头
    new_content = file.read()    # 读取文件中的新内容
    print(new_content)           # 输出文件中的新内容
~~~
---


### 复制文件内容写入新的文件
~~~python
# 当一行代码过长时可以使用 \ 来进行换行
with open("高能图片.webp", mode="rb") as f1, \
     open("高能图片复制体.webp", mode="ab") as f2:
    for line in f1:
        f2.write(line)
~~~
---

### 其他文件操作 
~~~python
# 引入系统函数库
import os 

# 创建目录
os.mkdir("aaa")

# 删除目录
os.rmdir("sss")

# 文件改名
os.rename("高能图片.webp","xxx.webp")

# 删除文件
os.remove("example.txt")
~~~
---