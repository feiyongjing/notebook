Stream：一个“流”
---------------------------------
Stream的API-创建Stream
Collection.stream()：Collection创建Stream
Stream.of()：数组创建Stream
String.chars()：字符串创建Stream
IntStream.range()：两个数之间的所有int类型的集合创建Stream(如果两数都是int类型，则包括第一个数不包括第二个数)
--------------------------------
Stream的API-中间操作
仍然返回Stream的操作
filter：对象过滤筛选操作
map：对象映射操作
sorted：对象排序操作
--------------------------------
Stream的API-终结操作
返回非‘Stream’的操作，包括void
一个流只能被消费一次
forEach：接受一个int不返回任何值（即void）
count：统计个数返回long(从虚空变出一个long)
max
min
findFirst：返回第一个对象（从虚空变出一个对象）
findAny：随机返回一个对象（从虚空变出一个对象）
anyMatch：返回一个布尔值（从虚空变出一个布尔值）
noneMatch：返回一个布尔值（从虚空变出一个布尔值）
-----------------------------------------
collector与Collectors
collect()流的收集操作
collect操作是最强大的操作
collect(Collectors.toList)    将一个流收集成一个List集合并返回
collect(Collectors.toSet)    将一个流收集成一个Set集合并返回
collect(Collectors.toMap)    将一个流收集成一个Map集合并返回
collect(Collectors.toCollection(具体实现))      将一个流收集成一个Collection
collect(Collectors.groupingBy(具体实现))      将一个流按照某个东西进行分类，按照分类返回一个Map集合
collect(Collectors.joinong(字符串))                将一个字符串流按照具体的字符串连接起来，返回一个字符串
collect(Collectors.summingInt(具体的int数据应用或其他))   输入一个int流返回其总和
-------------------------------------------------
并发流
parallelStream()
可以通过并发提高互相独立的操作性能
在正确使用的前提下，可以获得近似线性的提升
注意使用要小心，性能要测试，保证线程安全

