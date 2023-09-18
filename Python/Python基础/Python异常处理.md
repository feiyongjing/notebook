

~~~python
s = "abdiuhbdu"

# 使用 try 关键字确定可能出现异常的代码块
# 使用 except 关键字确定可能出现异常并且处理该异常
try:
    a = s[20]
except IndexError :
    print("处理IndexError")
~~~
---