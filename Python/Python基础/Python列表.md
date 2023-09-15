### []表示列表，内部可以装任何东西

### 列表的索引和截取
~~~python
# 创建空的list
lst = ["Java", "JavaScript", "C", "C++", "Python"]

# 通过索引获取列表中的元素，如果是负数就是从后向前获取元素，但是无法超过列表的边界否则抛出异常
print(lst[1])

# 列表的截取
# s[start:end]
# step表示步长，默认不写是1，正数代表从左向右截取列表，负数表示从右向左截取列表，并且数字表示每几个元素中取开头的元素
# step如果是正数那么start的索引必须在end索引左边，step如果是负数数那么start的索引必须在end索引右边
# 截取的列表包含start索引位置的元素但是不包含end索引位置元素
# 如果start省略就是截取列表开头到end索引位置元素的列表
# 如果end省略就是截取start索引位置到列表的结尾的子列表
# 如果是 s[:] 就还是原来的列表
print(lst[2:5])

lst1= ["x","q","g","7","b","d","8","1","0","g","h","7","e","0","2","1","t","y","e","0","2","1","e","7","y"]
print(lst1[-2:5:-3])

# 列表的循环
print("列表直接循环遍历")
for item in lst:
    print(item)

print("列表通过索引循环遍历")
for i in range(len(lst)):
    print(lst[i])


~~~
---

### 列表的增删改查
~~~python
lst = ["Java", "JavaScript", "C", "C++", "Python"]

# append()函数添加一个元素到列表末尾
lst.append("c#")

# extend()函数将参数列表的元素添加到列表末尾
lst.extend(["Shell", "Objective-C"])

# insert()函数添加元素到列表的指定索引位置，后面的元素全部向后移动一个索引位置
lst.insert(0,"Ruby")

print(lst)

# pop()函数删除指定索引的元素并且返回被删除的元素
print(lst.pop(7))

# remove()函数删除指定的元素
lst.remove("Objective-C")
print(lst)

# 修改列表索引位置的元素
lst[0]="php"
print(lst)

# 查询列表索引位置的元素
print(lst[1])

~~~
---


### 列表的排序
~~~python
lst = ["257", "241", "113", "132", "232"]

# 默认的排序
print(lst)

# 生序排序
lst.sort()
print(lst)

# 降序排序
lst.sort(reverse=True)
print(lst)

~~~
---