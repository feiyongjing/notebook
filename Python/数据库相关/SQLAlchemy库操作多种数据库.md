# SQLAlchemy 简介
- SQLAlchemy 是一个开源的 Python SQL 工具包，提供了一种灵活、高效的数据库访问方式。它支持多种数据库后端，包括 SQLite、MySQL、PostgreSQL 等，并且可以通过 ORM（对象关系映射）的方式操作数据库。相比于直接使用 SQL 语句，SQLAlchemy 提供了更加 Pythonic 的方式进行数据库操作。

## 安装sqlalchemy（笔记中sqlalchemy使用的版本是2.0.31）
~~~shell
pip install sqlalchemy
~~~

## sqlalchemy原生的核心API进行操作数据库
~~~python
from sqlalchemy import create_engine
from sqlalchemy.ext.asyncio import create_async_engine

from sqlalchemy import Table, Column, Integer, String, MetaData, text
from sqlalchemy import insert, update, delete, select, bindparam, Result

# 创建同步连接引擎，引擎通常是仅为特定数据库服务器创建一次的全局对象
engine = create_engine(
    # mysql同步连接字符串 f"mysql+mysqldb://{user}:{password}@{host}:{port}/{dbName}"
    str(f"mysql+mysqldb://root:123456@localhost:3306/testdb")
)

# 创建异步连接引擎
async_engine = create_async_engine(
    # mysql异步连接字符串 f"mysql+aiomysql://{user}:{password}@{host}:{port}/{dbName}"
    str(f"mysql+aiomysql://root:123456@localhost:3306/testdb")
)

def create_user_table() -> Table:
    # 创建数据表方式一：先创建数据表元数据结构再通过引擎创建数据表
    metadata = MetaData()
    users: Table = Table('users', metadata,
                  Column('id', Integer, primary_key=True, autoincrement=True),
                  Column('name', String(10)),
                  Column('age', Integer)
                  )
    metadata.create_all(engine)

    # 创建数据表方式二：直接执行sql
    # with engine.begin() as conn:
    #     conn.execute(text("CREATE TABLE users (id int primary key AUTO_INCREMENT, name varchar(10), age int)"))

    return users

def explicit_transaction():
    # 显式事务：需要显示提交事务
    # 注意显式事务并不会影响到表的创建，CREATE TABLE语句时，不需要显示提交也可以建表成功。甚至在建表语句后跟一个主键冲突无法成功执行的写入语句，在回滚时也不会把建表操作撤销
    # engine.connect()获取的连接不会自动提交事务，需要手动调用数据库连接的commit()方法提交事务
    # 可以划分为多段的sql，多次提交事务
    try:
        with engine.connect() as conn:
            conn.execute(text("CREATE TABLE some_table (x int primary key, y int)"))
            conn.execute(
                text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
      [{"x": 1, "y": 1}, {"x": 2, "y": 4}],
            )
            conn.commit()
            print("成功")
    except:
        conn.rollback()
        print("失败，回滚")

def implicit_transaction():
    # 隐式事务：预先将“连接”块声明为事务块，在“连接”块结束时隐式自动提交事务
    # engine.connect()获取的连接不会自动提交事务，需要手动调用数据库连接的commit()方法提交事务
    # 可以划分为多段的sql，多次提交事务
    try:
        with engine.begin() as conn:
            conn.execute(text("CREATE TABLE some_table (x int primary key, y int)"))
            conn.execute(
                text("INSERT INTO some_table (x, y) VALUES (:x, :y)"),
                [{"x": 1, "y": 1}, {"x": 2, "y": 4}],
            )
    except Exception as e:
        print(f"出错，需要手动编写回滚操作, {e}")

def insert_user(users: Table) -> None:
    # users表新增用户：先创建好需要新增的insert语句，然后通过引擎获取数据库连接执行语句
    insert_sql = users.insert().values(name='John', age=25)
    # 打印生成的sql
    print(insert_sql)
    with engine.begin() as conn:
        # 方式一：通过API生成sql执行或者直接生成sql同时传入新增的数据
        conn.execute(insert_sql)
        # conn.execute(insert(users), [{"name": 'John', "age": 25}])
        # 方式二：直接编写sql，并且通过传递绑定参数避免sql注入
        # conn.execute(
        #     text("INSERT INTO users (name, age) VALUES (:name, :age)"),
        #     [{"name": 'John', "age": 25}]
        # )

def update_user(users: Table) -> None:
    # users表更新用户：先创建好需要修改的update语句，然后通过引擎获取数据库连接执行语句
    update_sql = users.update().where(users.c.name == 'John').values(age=30)
    # 打印生成的sql
    print(update_sql)
    with engine.begin() as conn:
        # 方式一：通过API生成sql执行或者直接生成sql同时传入更新条件和数据
        # 注意bindparam函数中绑定的参数名称不能于数据表列名相同
        conn.execute(update_sql)
        # conn.execute(
        #     update(users).where(users.c.name == bindparam("b_name")).values(age=bindparam("b_age")),
        #     [{"b_name": 'John', "b_age": 30}]
        # )
        # 方式二：直接编写sql，并且通过传递绑定参数避免sql注入
        # conn.execute(
        #     text("UPDATE users SET age = :age WHERE name = :name"),
        #     [{"name": 'John', "age": 30}]
        # )

def delete_user(users: Table) -> None:
    # users表删除用户：先创建好需要删除的delete语句，然后通过引擎获取数据库连接执行语句
    delete_sql = users.delete().where(users.c.name == 'John')
    # 打印生成的sql
    print(delete_sql)
    with engine.begin() as conn:
        # 方式一：通过API生成sql执行或者直接生成sql同时传入删除条件数据
        # 注意bindparam函数中绑定的参数名称不能于数据表列名相同
        conn.execute(delete_sql)
        # conn.execute(delete(users).where(users.c.name == bindparam("b_name")), [{"b_name": 'John'}])
        # 方式二：直接编写sql，并且通过传递绑定参数避免sql注入
        # conn.execute(
        #     text("DELETE FROM users WHERE name = :name"),
        #     [{"name": 'John'}]
        # )

def select_user(users: Table) -> None:
    # 常见的条件查询方式
    # where中and连接多个表达式有两种方式
    # 方式1是链式调用where方法添加多个条件，例子：users.select().where(users.c.name == "John").where(users.c.age == 18)
    # 方式2是添加多个参数条件即默认是and，例子：users.select().where(users.c.name == "John", users.c.age == 18)
    # where中and和or组合使用，查询名称是张三或李四并且年龄等于18岁的用户例子如下
    # 例子：users.select().where( and_( or_(users.c.name == "张三", users.c.name == "李四"), users.c.age == 18))

    # 分组、分组过滤、排序查询
    # group by 排序例子：users.select().group_by(users.c.age))
    # having 分组后过滤例子：users.select().group_by(users.c.age)).having(users.c.name == "John")
    # order by 排序例子：users.select().order_by(users.c.id.desc()))

    # 分页查询
    # limit 设置查询数量和 offset 设置查询偏移量，查询第11到20条的数据例子：users.select().limit(10).offset(10)

    result: Result
    # users表查询用户：先创建好需要查询的select语句，然后通过引擎获取数据库连接执行语句
    # 默认是查询数据表的全部列，可以手动指定查询哪些列
    select_sql = users.select().where(users.c.name == "John").limit()
    # 打印生成的sql
    print(select_sql)
    with engine.begin() as conn:
        # 方式一：通过API生成sql执行或者直接生成sql同时传入查询条件数据
        # 注意bindparam函数中绑定的参数名称不能于数据表列名相同
        result = conn.execute(select_sql)
        # 查询指定的列，也可以写成 select(users.c["name", "age"])
        # result = conn.execute(select(users.c.name, users.c.age).where(users.c.name == bindparam("b_name")), [{"b_name": 'John'}])
        # 方式二：直接编写sql，并且通过传递绑定参数避免sql注入
        # result = conn.execute(
        #     text("SELECT * FROM users WHERE name = :name"),
        #     [{"name": 'John'}]
        # )
    # 打印查询的数据
    for user in result:
        print(f"name: {user.name}  age: {user.age}")


if __name__ == "__main__":
    users :Table =create_user_table()  # 测试数据表创建
    # explicit_transaction()  测试显式事务
    # implicit_transaction()  测试隐式事务

    # insert_user(users)     #测试insert新增用户
    # select_user(users)     #测试select查询用户
    # update_user(users)     #测试update更新用户
    # delete_user(users)     #测试delete删除用户
~~~

## sqlalchemy orm操作数据库
~~~python
from sqlalchemy import create_engine
from sqlalchemy.ext.asyncio import create_async_engine

from sqlalchemy import Table, Column, Integer, BigInteger, String, Boolean, DateTime, func, text, MetaData
from sqlalchemy import insert, update, delete, select, bindparam, Result

from sqlalchemy.orm import Session
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

from typing import Generator
from datetime import datetime

# 创建同步连接引擎，引擎通常是仅为特定数据库服务器创建一次的全局对象
engine = create_engine(
    # mysql同步连接字符串 f"mysql+mysqldb://{user}:{password}@{host}:{port}/{dbName}"
    str(f"mysql+mysqldb://root:cloudgrow.cn@192.168.0.119:30306/wz_shoe")
)

# 创建异步连接引擎
async_engine = create_async_engine(
    # mysql异步连接字符串 f"mysql+aiomysql://{user}:{password}@{host}:{port}/{dbName}"
    # str(f"mysql+aiomysql://root:123456@localhost:3306/testdb")
    str(f"mysql+aiomysql://root:cloudgrow.cn@192.168.0.119:30306/wz_shoe")
)

def get_db() -> Generator[Session, None, None]:
    # 迭代器迭代创建数据库的 session 会话，其实 session 会话内部引用了一个 Connection 数据连接
    # session 会话还包装了一个事务，这个事务将一直保持打开状态，直到会话提交或回滚
    with Session(engine) as session:
        yield session

# 创建声明式基类继承DeclarativeBase
class Base(DeclarativeBase):
    pass

# 数据表实体类继承声明式基类
class User(Base):
    __tablename__ = "user"                    # 数据表的名称设置
    __table_args__ = {"comment": "用户表"}    # 数据表的一些参数，comment是数据表的备注

    id: Mapped[int] = mapped_column(BigInteger, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(10), comment="用户名称")
    age: Mapped[int] = mapped_column(Integer, comment="用户年龄")
    is_deleted: Mapped[bool | None] = mapped_column(Boolean, default=False, comment="软删除")
    created_at: Mapped[datetime | None] = mapped_column(
        DateTime,
        default=func.now(),
        server_default=func.now(),
        nullable=True,
        comment="创建时间",
    )
    updated_at: Mapped[datetime | None] = mapped_column(
        DateTime,
        default=func.now(),          # 在客户端（Python 代码中）执行插入新记录时，如果该列未提供值，则使用 default 指定的默认值
        onupdate=func.now(),         # 在客户端（Python 代码中）执行更新记录时，使用 onupdate 指定的值更新该列
        server_default=func.now(),   # 在数据库服务器上执行插入新记录时，如果该列未提供值，则使用 server_default 指定的默认值
        server_onupdate=func.now(),  # 在数据库服务器上执行更新记录时，使用 server_onupdate 指定的值更新该列
        nullable=True,               # 指定该列是否可以为空
        comment="更新时间",           # 数据库列的备注
    )

    def __init__(self, name, age):
        self.name = name
        self.age = age


# 根据Base的继承关系在数据库创建继承Base类的所有实体类数据表
def create_table():
    Base.metadata.create_all(engine)

def insert_user(session: Session) -> None:
    try:
        # users表新增用户
        user : User=User("张三", 18)
        session.add(user)

        # users表批量新增用户
        users = [
            User("李四", 19),
            User("王五", 20),
            User("赵六", 21)
        ]
        session.add_all(users)
        session.commit()
    except:
        session.rollback()
        print("失败，回滚")

def update_user(session: Session) -> None:
    # 更新用户
    stmt = update(User).where(User.name.in_(["李四", "王五"])).values(age="16")
    session.execute(stmt)

    # 根据ID进行批量更新
    # session.execute(
    #     update(User),
    #     [
    #         {"id": 1, "name": "张三", "age": 26},
    #         {"id": 2, "name": "李四", "age": 13},
    #     ],
    # )
    session.commit()

def delete_user(session: Session) -> None:
    # 删除用户
    # stmt = delete(User).where(User.name.in_(["李四", "王五"]))
    # session.execute(stmt)

    # 根据ID进行批量删除，synchronize_session=False 参数可以提高性能，因为避免了在删除操作后同步会话状态
    session.execute(delete(User).where(User.id.in_([1, 2])))
    # session.query(User).filter(User.id.in_([1, 2])).delete(synchronize_session=False)
    session.commit()

def select_user(session: Session) -> None:
    # 控制查询结果需要的列，例子：session.query(User.name)
    # 条件过滤where查询，例子：session.query(User).filter(User.name == '小华')
    # group by 排序例子：sesssion.query(User).group_by(User.age))
    # having 分组后过滤例子：sesssion.query(User).group_by(User.age)).having(User.name == "John")
    # order by 排序，降序例子：sesssion.query(User).order_by(User.age.desc())
    # 分页查询，每页5条数据查询第3页数据例子：session.query(User).[11:5]
    # 内置SQL函数，求和例子：session.query(func.sum(User.age)).scalar()

    query = session.query(User)       # session.query()的返回值是Query对象，不能直接使用它的返回值作为查询结果
    # query.count()                   # 获取查询结果的数量，会触发执行SQL查询
    # query.all()                     # 返回查询结果的list，会触发执行SQL查询
    # query.get("123")                # 根据主键查询单个对象，会触发执行SQL查询
    # query.as_scalar()               # 返回此次查询的SQL语句

    # 遍历Query对象，也会触发执行SQL查询，遍历的是SQL查询的结果
    for user in query:
        print(f"name: {user.name}  age: {user.age}")



if __name__ == "__main__":
    users: Table = create_table()  # 测试数据表创建

    with next(get_db()) as session:
        insert_user(session)     #测试insert新增用户
        # select_user(session)     #测试select查询用户
        # update_user(session)     #测试update更新用户
        # delete_user(session)     #测试delete删除用户
~~~



