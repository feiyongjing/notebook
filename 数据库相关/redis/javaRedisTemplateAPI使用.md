参考：https://www.jianshu.com/p/0fa4c100e9a9

### 直接使用的方法
~~~java
Boolean delete(K key);    // 删除单个的key, 成功返回true

Long delete(Collection<K> keys);   // 删除一组key, 返回删除的数量

Boolean expire(K key, long timeout, TimeUnit unit);  //指定key的有效期，存入后在timeout时间内有效，unit时间单位

Long getExpire(K key);  // 获取key的有效时间

Long getExpire(K key, TimeUnit timeUnit); // 获取key的有效时间, 并转换为以timeUnit为时间单位

Boolean hasKey(K key); // 判断key是否存在
~~~

以下的不同数据类型操作的两种API的方法基本类似，所以第二种方式简单的写了一下

### String类型相关的操作方法

~~~java
 //1、通过BoundValueOperations设置值
BoundValueOperations stringKey = redisTemplate.boundValueOps("StringKey");  //获取与指定key绑定的BoundValueOperations对象
stringKey.get();                    // 获取绑定键的值
stringKey.get(0，5);                // 获取指定区间的值   
stringKey.set("StringVaule");       // 给绑定键重新设置值(如果没有值，则会添加这个值)
stringKey.set("StringValue",1, TimeUnit.MINUTES); // 给绑定键设置新值并设置过期时间  
stringKey.expire(1, TimeUnit.MINUTES);            // 设置绑定键的过期时间
stringKey.getAndSet("StringValue"); // 设置绑定键的值并返回其旧值。
stringKey.append("添加的value");    // 在原来值的末尾添加值
stringKey.size();                   // 获取绑定键的值长度

//2、通过ValueOperations设置值
ValueOperations ops = redisTemplate.opsForValue();
ops.set("StringKey", "StringVaule");
ops.set("StringValue","StringVaule",1, TimeUnit.MINUTES);


~~~
 

### Hash类型的操作

~~~java
//1、通过BoundHashOperations设置值
BoundHashOperations hashKey = redisTemplate.boundHashOps("HashKey");
hashKey.put("SmallKey", "HashVaue");  // 添加键值对
hashKey.putIfAbsent("Smallkey1", "HashValue1")    // 如果map键不存在，则新增，存在，则不变
hashKey.getKey();                     // 获取绑定键的值，在当前是 “Hashkey”
hashKey.values();                     // 获取所有的值
hashKey.entries();                    // 获取整个Map的键值对
hashKey.keys();                       // 获取所有的键
hashKey.get("Smallkey");              // 获取Map中指定键对应的值
hashKey.size();                       // 获取键值对的数量

Map map = new HashMap<>();
map.put("m1","n1");
map.put("m2","n2");
hashKey.putAll(map);                 // 批量新增键值对


List list = new ArrayList<>(Arrays.asList("ww","w1"));
hashKey.multiGet(list)               // 批量获取键对应的值

hashKey.delete("m1", "m2")           // 删除指定键，接收的是数组可以批量删除


//2、通过HashOperations设置值
HashOperations hashOps = redisTemplate.opsForHash();
hashOps.put("HashKey", "SmallKey", "HashVaue");  // 添加键值对
hashOps.get("HashKey", "SmallKey")               // 获取键对应的值
~~~
 

### Set类型相关操作

~~~java
//1、通过BoundSetOperations设置值
BoundSetOperations setKey = redisTemplate.boundSetOps("setKey");
setKey.add("setValue1", "setValue2", "setValue3");   // 批量添加值
setKey.members();                                    // 获取全部的值
setKey.isMember("setValue2");                        // 判断指定的值是否存在
Object o = setKey.randomMember();                    // 随机获取一个值
List list = setKey.randomMembers(5);                 // 随机获取指定数量的值
setKey.pop();                                        // 随机删除一个值，并返回指
setKey.remove("a1", "a2");                           // 批量删除值
setkey.size();                                       // 获取值的个数

//2、通过SetOperations设置值
SetOperations setOps = redisTemplate.opsForSet();
setOps.add("setKey", "SetValue1", "setValue2", "setValue3");    // 批量添加值
setOps.members("setKey")                                        // 获取全部的值
~~~
 

### LIST类型相关操作

~~~java
//1、通过BoundListOperations设置值
BoundListOperations listKey = redisTemplate.boundListOps("listKey");
listKey.leftPush("listLeftValue3");     // 在左边添加值
listKey.rightPush("listRightValue4");   // 在右边添加值
listKey.leftPushAll("a1","a2","a3");    // 在左边批量添加值  
listKey.rightPushAll("a1","a2","a3")    // 在右边批量添加值
listKey.leftPush("a1", "1");            // 在第一个值是a1的左边添加1
listKey.rightPush("a2", "2");           // 在第一个值是a2的右边添加2
listKey.leftPop("a1");                  // 在左边添加值
listKey.rightPop("a2");                 // 在右边添加值
listKey.leftPop(1, TimeUnit.SECONDS);   // 1秒后在左边添加值
listKey.rightPop(1, TimeUnit.SECONDS);  // 1秒后在右边添加值
listKey.range(0.-1);                    // 返回下标区间的值，-1值末尾
listKey.index(5);                       // 返回指定下标对应的值
listKey.set(0,"aaa");                   // 在指定的下标添加值
listKey.remove(4,"aaa");                // 移除指定个数的值，即aaa

//2、通过ListOperations设置值
ListOperations opsList = redisTemplate.opsForList();
opsList.leftPush("listKey", "listLeftValue5");
opsList.rightPush("listKey", "listRightValue6");
~~~
 

### Zset类型(有序集合)的相关操作

~~~java
//1、通过BoundZsetOperations设置值
BoundZSetOperations zSetKey = redisTemplate.boundZSetOps("zSetKey");
zSetKey.add("zSetVaule", 100D);  // 添加值并设置分数（分数是Double类型），内部按分数从小到大维持排序

DefaultTypedTuple<String> p1 = new DefaultTypedTuple<>("zSetVaule1", 2.1D);
DefaultTypedTuple<String> p2 = new DefaultTypedTuple<>("zSetVaule2", 3.3D);
zSetKey.add(new HashSet<>(Arrays.asList(p1,p2)));  // 批量添加值和分数

zSetKey.range(0, -1);       // 获取指定区间的值，-1代表区间末尾
zSetKey.score("zSetVaule"); // 获取指定值的分数
zSetKey.size();             // 获取值的个数
zSetKey.count(0D, 2.2D);    // 返回指定分数区间的值个数

//2、通过ValueOperations设置值
ZSetOperations zSetOps = redisTemplate.opsForZSet();
zSetOps.add("zSetKey", "zSetVaule", 100D);
~~~