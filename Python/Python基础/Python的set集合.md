# set集合元素是无序、不可重复的，set存储的元素数据类型必须是可以进行哈希计算的
- 可以进行哈希计算的数据类型都是不可变的，例如 int、bool、tuple(元组，元组中的元素数据类型也必须是不可变的)、str(字符串)
- 不可以进行哈希计算的数据类型都是可变的，例如 list(列表)、dict(字典)、set
~~~python
# 注意如果s = {} 中没有元素那么变量s是字典dict类型，而如何需要空的set请使用 set()函数

s={'23',123,"12w2"}


print(type(s))


# 添加元素
s.add("1w3ght")
print(s)

# 随机删除一个元素，并且返回该元素
print(s.pop())
print(s)

# 删除一个指定的元素
s.remove("23")
print(s)

# 没有直接修改函数，想要修改就删除旧数据再添加新的数据

# 循环遍历
for item in s:
    print(item)

~~~
---

### 交集、并集、差集
~~~python
s1 = {111, 222, 333}
s2 = {222, 333, 444}

# 交集的获取
print(s1 & s2)
print(s1.intersection(s2))

# 并集的获取
print(s1 | s2)
print(s1.union(s2))

# 差集的获取
print(s1 - s2)
print(s1.difference(s2))
~~~
---