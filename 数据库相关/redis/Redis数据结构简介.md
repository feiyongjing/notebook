常见的5种数据结构：String   List  Set  Hash  zSet 

Redis的数据结构本质上是一个Map，所以5种数据结构相当于

~~~
String //相当于Java里的 Map<String, String>

List   //相当于Java里的 Map<String, List<Object>>

Set    //相当于Java里的 Map<String, Set<Object>>
 
Hash   //相当于Java里的 Map<String, HashMap<Object, Object>>

zSet   //相当于Java里的 Map<String, LinkedHashSet<Object>>
~~~
 