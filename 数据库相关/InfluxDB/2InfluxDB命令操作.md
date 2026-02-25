#  influxdb3的CLI参考（influxdb3没有交互式的命令行）
https://docs.influxdb.org.cn/influxdb3/core/reference/cli/influxdb3/

## 数据库操作
~~~sql
-- 显示数据库
show databases

-- 创建数据库
create database [数据库名称] 

-- 删除数据库
drop database [数据库名称] 

-- 切换使用数据库
use [数据库名称]
~~~

## 数据保留策略设置
~~~sql
-- 保留策略语法
CREATE RETENTION POLICY [保留策略的名称：这个名称是自定义的] ON [数据库名称] DURATION [该保留策略对应的数据过期时间] REPLICATION [副本因子：多节点的情况下设置，单节点都是1] [SHARD DURATION <分片组默认时长>] [是否为默认策略]

-- 创建数据保留策略
CREATE RETENTION POLICY "influx_retention" ON [数据库名称] DURATION 30d REPLICATION 1 DEFAULT:

-- 查看保留期
SHOW RETENTION POLICIES ON [数据库名称]

-- 修改保留期(如下是15天)
ALTER RETENTION POLICY "influx_retention" ON [数据库名称] 15d

-- 删除保留期
DROP RETENTION POLICY "influx_retention" ON [数据库名称]
~~~

## 表操作
~~~sql
-- 显示所有表
show measurements

-- 插入数据，若表不存在会直接创建表并且插入数据
insert [表名],altitude=1000,area=北,temperature=11,humidity=-4

-- 查询表，InfluxDB 3.x 原生支持标准 SQL 查询（PostgreSQL 兼容）
select * from [表名]

-- 设置一下时间显示格式
precision rfc3339

-- 删除表
DROP MEASUREMENT [表名]
~~~

















