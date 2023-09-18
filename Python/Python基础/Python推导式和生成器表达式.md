### 推导式
~~~python

# list列表推导式   [元素数据 for循环  if条件]
# set集合推导式    {元素数据 for循环  if条件}
# dict字典推导式   {k:v      for循环 if条件}
# 注意没有tuple元组推导式

lst = ["adw", "xqewxe", "xqwegf"]

# 例子1：将list中的元素转换为大写放入新的list
print([item.upper() for item in lst])

# 例子2：将list中的元素长度大于4的筛选出来放入新的list
print([item for item in lst if len(item) > 4])

lst1 = ["琴瑟仙女 娑娜  ", "    琴瑟仙女 娑娜", " 迅捷斥候 提莫 "]
# 例子3：将list中的元素去除首尾空格后放入新的set
print({item.strip() for item in lst1})

lst2 = ["打野恶魔小丑萨科", "辅助琴瑟仙女娑娜", "AD寒冰射手艾希"]
# 例子5：将list中的元素按照阵容分配后放入新的dict字典
print({item[0:2]: item[2:] for item in lst2})

~~~
---


### 生成器表达式
~~~python

# 生成器表达式   (元素数据 for循环  if条件)
lst2 = ["打野恶魔小丑萨科", "辅助琴瑟仙女娑娜", "AD寒冰射手艾希"]

# 获取生成器，本质是迭代器
gen = (item for item in lst2)
print(type(gen))

# 进行迭代获取数据，或者直接放进list()函数的参数中转换为list
for item in gen:
    print(item)

~~~
---