# docker安装PostgreSQL
~~~shell
# 启动PostgreSQL，设置端口、数据文件挂载、用户名密码、预先创建业务数据库、时区设置， 请选择LTS版本（偶数版本是5年维护期，奇数版本只有1年维护期）
docker run -d --name pg-web --restart always -p 127.0.0.1:5432:5432 -v /data/postgres/data:/var/lib/postgresql/data -e POSTGRES_USER=appuser -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=appdb -e TZ=Asia/Shanghai postgres:16  


docker run -d --name pg-web --restart always -p 127.0.0.1:5432:5432 -v 'E:\project\test\postgresql-test-data':/var/lib/postgresql/data -e POSTGRES_USER=appuser -e POSTGRES_PASSWORD=123456 -e POSTGRES_DB=appdb -e TZ=Asia/Shanghai postgres:16  

# 进入容器连接数据库
docker exec -it pg-web bash
psql -h 127.0.0.1 -p 5432 -U appuser -d appdb
# 或者不进入容器直接连接
docker exec -it pg-web psql -h 127.0.0.1 -p 5432 -U appuser -d appdb
~~~

## PostgreSQL中常见的数据类型 
### 数字类型
~~~
smallint	        2 字节	小范围整数	            -32768 到 +32767
integer	            4 字节	常用的整数	            -2147483648 到 +2147483647
bigint      	    8 字节	大范围整数	            -9223372036854775808 到 +9223372036854775807
decimal	            可变长	用户指定的精度，精确	 小数点前 131072 位；小数点后 16383 位
numeric	            可变长	用户指定的精度，精确	 小数点前 131072 位；小数点后 16383 位
real        	    4 字节	可变精度，不精确	     6 位十进制数字精度
double precision	8 字节	可变精度，不精确	     15 位十进制数字精度
smallserial	        2 字节	自增的小范围整数	     1 到 32767
serial	            4 字节	自增整数	            1 到 2147483647
bigserial	        8 字节	自增的大范围整数	     1 到 9223372036854775807
~~~

### 货币类型
money 类型存储带有固定小数精度的货币金额。
numeric、int 和 bigint 类型的值可以转换为 money，不建议使用浮点数来处理处理货币类型，因为存在舍入错误的可能性。
~~~
money	8 字节	货币金额	-92233720368547758.08 到 +92233720368547758.07
~~~

### 字符类型
~~~
character varying(n), varchar(n)    变长，有长度限制
character(n), char(n)               f定长,不足补空白
text                                变长，无长度限制
~~~

### 日期/时间类型
~~~
名字	存储空间	描述	                 最低值	  最高值	         分辨率
timestamp [ (p) ] [ without time zone ]	    8 字节	日期和时间(无时区)	4713 BC到294276 AD精确到1 毫秒 / 14 位
timestamp [ (p) ] with time zone	        8 字节	日期和时间，有时区	4713 BC到294276 AD精确到1 毫秒 / 14 位
date	                                    4 字节	只用于日期	4713 BC到5874897 AD精确到1 天
time [ (p) ] [ without time zone ]	        8 字节	只用于一日内时间	00:00:00到24:00:00精确到1 毫秒 / 14 位
time [ (p) ] with time zone	                12 字节	只用于一日内时间，带时区	00:00:00+1459到24:00:00-1459精确到1 毫秒 / 14 位
interval [ fields ] [ (p) ]	                12 字节	时间间隔	-178000000年到178000000年精确到1 毫秒 / 14 位
~~~

### 枚举类型
- 枚举类型可以用于表和函数定义
~~~sql
-- 定义方式如下
CREATE TYPE mood AS ENUM ('sad', 'ok', 'happy');
CREATE TYPE week AS ENUM ('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun');

-- 枚举被用于定义表字段
CREATE TABLE person (
    name text,
    current_mood mood
);
~~~

### 网络地址类型
PostgreSQL 提供用于存储 IPv4 、IPv6 、MAC 地址的数据类型。
用这些数据类型存储网络地址比用纯文本类型好， 因为这些类型提供输入错误检查和特殊的操作和功能。
~~~
cidr	7 或 19 字节	IPv4 或 IPv6 网络
inet	7 或 19 字节	IPv4 或 IPv6 主机和网络
macaddr	6 字节	        MAC 地址
~~~

### JSON 类型
json 数据类型可以用来存储 JSON（JavaScript Object Notation）数据， 这样的数据也可以存储为 text，但是 json 数据类型更有利于检查每个存储的数值是可用的 JSON 值。

### 数组类型
PostgreSQL 允许将字段定义成变长的多维数组。
数组类型可以是任何基本类型或用户定义类型，枚举类型或复合类型。
~~~sql
-- 举例
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[],
    schedule        text[][]
);
~~~








