### dict字典是以键值对存储数据的
~~~python
# key的数据类型必须是可以哈希的类型，value的数据类型可以是任意数据类型
# 如下列表作为key是不允许的
# dic1={[]:"axas"}

dic = {"吸血鬼": "弗拉基米尔", "寒冰射手": "艾希", "邪恶小法师": "维迦"}

print(dic)

# 获取全部的key
print(list(dic.keys()))

# 获取全部的value
print(list(dic.values()))


# 循环遍历字典，可以直接获取到key
for key in dic:
    print(key, dic[key])

# 循环遍历字典，可以直接获取到key和value
for key, value in dic.items():
    print(key, value)

# 循环遍历字典
for item in dic.items():
    key, value = item  # 列表或元组的解包，将里面的元素按照索引顺序赋值给对应的变量，如果变量的数量和元素的数量对不上会抛出异常
    print(key, value)

~~~

### dict字典的增删改查
~~~python
dic = {"吸血鬼": "弗拉基米尔", "寒冰射手": "艾希", "邪恶小法师": "维迦"}
print(dic)


# 新增key和value，如果key已经存在了就会覆盖key对应的value
dic["天启者"] = "卡尔玛"
print(dic)

# 如果key在字段中不存在就新增key和value，否则不进行操作
dic.setdefault("天启者", "asda")
dic.setdefault("狂野女猎手", "奈德丽")
print(dic)

# 删除key和对应的value
dic.pop("邪恶小法师")


# 查询元素，根据key直接获取value
print(dic["寒冰射手"])   # key不存在时会抛出异常
print(dic.get("吸血鬼")) # key不存在时会返回None，None就是空

~~~