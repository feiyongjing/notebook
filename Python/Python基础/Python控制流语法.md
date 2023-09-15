~~~python
# python不使用花括号表示作用域，而是使用: 和下一行的缩进表示作用域。
# 如果没有确定要执行的代码但是有if语句或者是循环语句会报错，可以使用关键字pass 临时替代条件控制或循环中的代码块，使代码运行不报错

# if 控制流
money=500
if money>100:
    if money>300:
        money-=300
        print("铁锅炖")
    else:
        money -= 100
        print("看电影")
elif money>1000:
    print("购物")
else:
    print("喝西北风")

print("回家")

# while 条件循环
print("while 条件循环0数到9")
j=0
while j<10:
   print(j)
   j+=1

# for 循环
# range(n) 表示从0数到n，注意n是正数
# range(n, m) 表示从n数到m，注意n和m都是正数
# range(n, m, q) 表示从n数到m，每一次的间隔是q，注意n、m、q都是正数
print("for 条件循环0数到9")
for i in range(10):
    print(i)

print("for 条件循环遍历字符串")
s="你好啊，我叫赛丽亚"
for c in s:
    print("遍历字符串的字符：",c)


# break 和 continue 分别表示完全退出上层循环 和 跳过本次循环进入下一次循环
~~~