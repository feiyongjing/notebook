# Redis连接命令
~~~shell
# 连接本地的Redis
redis-cli              
# 连接远程的Redis 命令结果通常是显示中如果是1表示成功，0表示失败，注意有一些是例外的需要其他的命令进行检查                                     
redis-cli -h host -p port -a password  
# 如果在登录时不指定密码，则登录后使用命令无权限，需要使用 auth myPassword 验证密码获得权限

# 也可以使用 redli 来连接redis，相比redis-cli来说 redli 更加支持windows，例如支持TLS加密连接
redli -h xxx -p 6379 --tls --certfile xxx.pem


# 切换数据区
select [num]

# 清除当前数据区的数据
flushdb
~~~

# Redis keys 命令
~~~shell
# key 存在时删除 key
del key                         

# 检查给定 key 是否存在
exists key                    

# 设置key过期时间，以秒计，以毫秒计使用pexpire
expire key number    

# 返回给定 key 的剩余生存时间，以秒为单位，以毫秒为单位使用pttl
ttl key                           

# 移除 key 的过期时间，key 将持久保持
persist key                   

# 查找所有符合给定正则匹配( pattern)的 key 
keys pattern                

# 将当前数据库的 key 移动到给定数字的数据库(分区) db 当中
move key dbNumber 

# 修改 key 的名称，如果新的名字已经存在就会覆盖值
rename key newkey    

# 仅当 newkey 不存在时，将 key 改名为 newkey 
renamenx key newkey  

# 返回 key 所储存的值的类型
type key                          

# 从当前数据库中随机返回一个 key
randomkey                     

# 序列化给定 key ，并返回被序列化的值
dump key                        
~~~

# Redis 字符串命令
~~~shell
# 增加一对键值对，如果键已经存在就覆盖值
set key value                                        

#  获取一个指定key的值
get key                                                  

# 增加多对键值对，如果键已经存在就覆盖值
mset key1 value1 key2 value2           

# 获取多个指定key的值
mget key1 key2                                     

# 只有在 key 不存在时设置 key 的值
setnx key value                                     

# 增加多对键值对，只有在 key 不存在时设置 key 的值
msetnx key1 value1 key2 value2       

# 覆盖key的旧值，并返回旧值
getset key value                                    

# 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)，以毫秒为单位使用psetex
setex key number value                      

# 用 value 覆写给定 key 所储存的字符串值，从偏移量 offset 开始(包含offset)
setrange ket offset value                    

# 根据start和end两个数字作为下标截取key对应的值，注意包含start和end，如果end是  -1则代表末尾
getrange key start end                         

# 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾
append key value                                 

# 返回 key 所储存的字符串值的长度
strlen key                                               

# 将 key 中储存的数字值增一
incr key                                                   

# 将 key 所储存的值加上给定的增量值（increment） 
incrby key increment                          

# 将 key 中储存的数字值减一
decr key                                                 

# key 所储存的值减去给定的减量值（decrement）
decrby key decrement                         

# 根据offset数字获取key对应值的指定位（1个字符占8个位）
getbit key offset                                    

# 根据offset数字设置key对应值的指定位的值（1个字符占8个位）
setbit key offset value                          

~~~

# redis 哈希  相当于JAVA的HashMap, key表示Map对象名字，field表示Map的键，value表示Map的值
~~~shell
# 向哈希表 key 中添加一个字段 field ，值设为 value
Hset key field value                                

# 获取存储在哈希表中一个指定字段的值
Hget key field                                           

# 向哈希表 key 中添加多个字段 field 和值value
Hmset key field1 value1 field2 value2  

# 获取存储在哈希表中多个指定字段的值
Hmget key field1 hield2                          

# 只有在字段 field 不存在时，设置哈希表字段的值
Hsetnx key field value                             

#删除一个或多个哈希表字段
Hdel key field1 field2                               

# 查看哈希表 key 中，指定的字段是否存在
Hexiste key field                                       

# 获取所有哈希表中的字段
Hkeys key                                                   

# 获取哈希表中所有值
Hvals key                                                    

# 获取在哈希表中指定 key 的所有字段和值
Hgetall key                                                 

# 获取哈希表中字段的数量
Hlen key                                                      

# 为哈希表 key 中的指定字段的整数值加上增量 increment 
Hincrby key field increment                    

# 为哈希表 key 中的指定字段的浮点数值加上增量 increment 
Hincrbyfloat key field increment            

~~~

# Redis List 命令
~~~shell
# 添加一个或多个值插入到列表头部，列表不存在创建列表进行添加
Lpush key value1 value2                            

# 添加一个或多个值插入到以存在的列表头部，列表不存在则不进行任何操作
Lpushx key value1 value2                       

# 添加一个或多个值插入到列表尾部，列表不存在创建列表进行添加
Rpush key value1 value2                         

# 添加一个或多个值插入到以存在的列表尾部，列表不存在则不进行任何操作
Rpushx key value1 value2                       

# 将值 value 插入到列表 key 当中，位于值 pivot 之前
Linsert key before pivot value                

# 将值 value 插入到列表 key 当中，位于值 pivot 之后
Linsert key after pivot value                   

# 通过索引下标来替换指定列表元素的值，负数索引下标代表从倒数第几个开始数，索引下标不能超出列表的范围
Lset key index value                                

# 用于通过索引下标获取列表中的元素值，负数索引下标代表从倒数第几个开始数
Lindex key index                                      

# 返回列表中指定区间内的元素，区间以偏移量 start 和 end 指定，负数索引下标代表从倒数第几个开始数
Lrange key start end                               

# 返回列表的长度，列表不存在返回0
Llen key                                                     

# 移出并获取多个列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时时间number(单位秒)或发现可弹出元素为止
Blpop key1 key2 number                       

# 移出并获取多个列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时时间number(单位秒)或发现可弹出元素为止
Brpop key1 key2 number                      

# 用于移除并返回列表头部number个元素(number必须是正数)，不加number只移除1个元素
Lpop key number                                    

# 用于移除并返回列表尾部number个元素(number必须是正数)，不加number只移除1个元素
Rpop key number                                   

# 移除source列表的最后一个元素，并将该元素添加到destination列表的头部并返回
Rpoplpush source destination              

# 移除source列表的最后一个元素，并将该元素添加到destination列表的头部并返回，如果列表没有元素会阻塞列表直到等待超时时间number(单位秒)或发现可弹出元素为止
Brpoplpush source destination number            

# 移除列表中指定区间之外的全部元素，区间以偏移量 start 和 end 指定(区间包含这两个元素)，负数索引下标代表从倒数第几个开始数
Ltrim key start end                                 

# 根据参数 number 的值，移除列表中与参数 VALUE 相等的元素
Lrem key number value                        
# number > 0 : 从表头开始向表尾搜索，移除与 value 相等的元素，数量为 number 
# number < 0 : 从表尾开始向表头搜索，移除与 value 相等的元素，数量为 number 的绝对值。
# number = 0 : 移除表中所有与 value 相等的值。

~~~

# Redis Set 命令
~~~shell
# 向集合添加一个或多个元素
Sadd key value1 value2                        

# 查看集合元素数量
Scard key        

# 判断集合是否存在指定元素
Sismember key value                                

# 返回集合中的所有的元素， 不存在的集合 key 被视为空集合
Smembers key                                            

# 将member元素从source集合移动到destination集合
Smove source destination member       

# 随机移除指定集合number个元素
Spop key number                                      

# 随机查看指定集合number个元素
Srandmember key number                      

# 移除集合中的一个或多个指定的成员元素，不存在的成员元素会被忽略
Srem key value1 value2                            

# 查看只有第一个key存在而其他key中不存在的元素
Sdiff key1 key2                                             

# 查看只有第一个key存在而其他key中不存在的元素，并存入destination集合，如果 destination 集合已经存在，则将其覆盖，destination 可以是 key 本身
Sdiffstore destination key1 key2              

# 返回给定所有给定集合的交集。 不存在的集合 key 被视为空集。 当给定集合当中有一个空集时，结果也为空集
Sinter key1 key2                                         

# 将给定集合之间的交集存储在指定的destination集合中。如果指定的集合已经存在，则将其覆盖
Sinterstore destination key1 key2          

# 返回多个集合的并集，不存在的集合 key 被视为空集
Sunion key1 key2                                       

# 返回多个集合的并集，并存入destination集合，如果 destination 已经存在，则将其覆盖。
Sunionstore destination key1 key2         

~~~

# Redis ZSet 命令
~~~shell
# 向有序集合添加一个或多个score分数和元素，元素以存在就更新分数并从插入新的存储位置
Zadd key score1 value1 score2 value2   

# 查看有序集合元素数量
Zcard key                                                     

# 计算在有序集合中指定区间分数的成员元素数
Zcount key min max                                 

# 有序集合中对指定成员元素的分数加上增量 increment
Zincrby key increment value                    

# 计算给定的一个或多个有序集的交集保存到destination有序集合中，其中给定 key 的数量必须以 numkeys 参数指定
Zinterstore destination numkeys key1 key2 

# 计算给定的一个或多个有序集的并集保存到destination有序集合中，其中给定 key 的数量必须以 numkeys 参数指定
Zunionstore destination numkeys key1 key2 

# 成员的位置先按分数值递增(从小到大)来排序，然后查看指定索引下标区间内的成员元素
Zrange key start end                                       

# 成员的位置先按分数值递增(从大到小)来排序，然后查看指定索引下标区间内的成员元素
Zrevrange key start end                                 

# 成员的位置先按分数值递增(从小到大)来排序返回有序集合，然后查看指定索引下标区间内的成员元素和分数
Zrange key start end withscores                  

# 成员的位置先按分数值递增(从大到小)来排序返回有序集合，然后查看指定索引下标区间内的成员元素和分数
Zrevrange key start end withscores            

# 返回有序集合中指定分数区间(min和max之间)的成员列表，有序集成员按分数值递增(从小到大)次序排列，并支持选择分页返回，偏移量offset，返回数量count
Zrangebyscore key min max  Limit offset count  

# 返回有序集合中指定分数区间(min和max之间)的成员列表，有序集成员按分数值递增(从大到小)次序排列，并支持选择分页返回，偏移量offset，返回数量count
Zrevrangebyscore key min max  Limit offset count  

# 返回有序集合中指定成员的分数
Zscore key value                                            

# 返回有序集合中指定成员的排名(分数从小到大的排名，最小的排名显示是0)
Zrank key value                                              

# 返回有序集合中指定成员的排名(分数从大到小的排名，最大的排名显示是0)
Zrevrank key value                                         

# 移除有序集合中的一个或多个成员元素
Zrem key value1 value2                                

# 按照start和end指定的索引区间（包含start和end），移除区间内的全部成员元素             
Zremrangebyrank key start end                                               

# 按照min和max指定的分数区间（包含min和max），移除区间内的全部成员元素 
Zremrangebyscore key min max                 

~~~

