# 字符串经过编码后得到不同的bytes编码
~~~python
s = "拉克丝"

# 字符串转换为gbk或者是utf-8的字节编码
bs1=s.encode("gbk")
bs2=s.encode("utf-8")

print(type(bs1))
print(bs1)
print(bs2)
~~~

# 将gbk编码转换为utf-8编码，反之一样的操作
~~~python
a = "光辉女郎"
a1=a.encode("gbk")
a2=a.encode("utf-8")

# 先将gbk编码解码转换成原始文字内容
s1 = a1.decode("gbk")
# 再将原始文字进行utf-8编码
s2 = s1.encode("utf-8")
print(a2==s2)
~~~