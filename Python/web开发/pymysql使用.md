### 使用pymysql
~~~python
# 安装pymysql包 pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pymysql
import pymysql

# 建立数据库连接
conn = pymysql.connect(
    host='localhost',
    port=30306, user='root',
    password='cloudgrow.cn',
    # db='testdb',
    # autocommit=True   是否自动提交新增编辑删除等操作的sql修改，默认是不开启的
)

# 创建游标对象
cursor = conn.cursor()

# 选择新的数据库
cursor.execute("USE `python-test`")

# 执行 SQL 查询
cursor.execute("SELECT * FROM test")

# 获取查询结果
results: tuple = cursor.fetchall()
for row in results:
    print(row)

# 注意新增编辑删除等操作的sql是需要通过数据库连接进行手动提交修改，除非在连接时设置了自动提交新增编辑删除等操作的sql修改
cursor.execute("insert into test (`name`, age) values ('寒冰射手艾希','19')")
# 手动提交sql修改
conn.commit()

# 注意不要使用字符串的拼接和f-string等拼接字符串，而是使用参数化查询，占位符的数量必须与查询参数的数量相同
# 以下是占位符参数化查询，可以避免sql注入，防止查出全部的数据
username = "'寒冰射手艾希' OR 1=1 --"
sql = "SELECT * FROM test WHERE name=%s"
cursor.execute(sql, (username,))
# 获取查询结果
results: tuple = cursor.fetchall()
for row in results:
    print(row)

# 关闭游标和数据库连接
cursor.close()
conn.close()

~~~
---