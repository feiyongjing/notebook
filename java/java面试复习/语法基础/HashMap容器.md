## 基本操作
HashMap容器实现了接口Map，从功能上讲，它是一个映射，可以实现通过键（key）快速获取值（value），示例代码如下所示。HashMap容器中的键和值具有一一映射关系，其中，键不可重复，但值可以重复。如果往HashMap容器中存储重复的键，那么，后存储的值会覆盖先前的值，相当于执行了修改操作。
~~~java
public class Demo13_1 {
  private static class Student {
    public int id;
    public int score;
    public Student(int id, int score) {
      this.id = id;
      this.score = score;
    }

    @Override
    public String toString() {
      return "Student [id=" + id + "; score=" + score + "]";
    }
  }

  public static void main(String[] args) {
    Map<Integer, Student> stuMap = new HashMap<>();
    // 增
    stuMap.put(1, new Student(1, 90));
    stuMap.put(3, new Student(3, 88));
    stuMap.put(19, new Student(19, 97));

    // 删
    stuMap.remove(1);

    // 改
    stuMap.put(3, new Student(3, 100));

    // 查
    Student stu = stuMap.get(3);
    System.out.println(stu); //Student [id=3; score=100]
  }
}
~~~

## hash冲突解决
HashMap容器是基于哈希表实现的。HashMap容器将键和值包裹为Node类对象存储在哈希表中。Node类的定义如下所示。在接下来的分析中，如果不特别说明，我们均按照JDK8中HashMap容器的代码实现来讲解。
~~~java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

  static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
    //...省略getter、setter等方法...
  }

  transient Node<K,V>[] table;
  //...省略其他属性和方法...
}
~~~

哈希表存在哈希冲突问题。解决哈希冲突问题的方法有多种，其中比较常用的方法是链表法（也叫做拉链法），这也正是HashMap容器所使用的方法。如上代码所示，table数组用来存储各个链表。Node类表示链表的节点。在Node类中，key表示键，value表示值，next为链表的next指针，hash为由key通过哈希函数计算得到的哈希值。在查询元素时，HashMap用此值做预判等。存在哈希冲突的节点会存储在同一个链表中。

## 哈希函数
哈希函数是哈希表中非常关键的组成部分。HashMap中的哈希函数如下代码所示。
~~~java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
~~~

从上述代码发现，HashMap的哈希函数设计得非常简单，将键的hashCode()函数的返回值h，与移位之后的h进行异或操作，最终得到的值就作为哈希值。
因为key经过此哈希函数计算之后，得到的哈希值跟n(哈希表的长度)取模，才能最终得到数据存储在table数组中的下标（我们把这个下标值简称为“key对应数组下标”）。因为取模操作比较耗时，所以，在具体实现时，Java使用位运算来实现取模运算，如下代码所示。key将存储在table[index]对应的链表中。
~~~java
int index = hash(key) & (n-1);
~~~
其中，HashMap容器的数组长度n的默认初始值为16。


上面粗略展示了key的哈希值和对应数组下标的计算方式

### key的hashCode()函数

hashCode()函数定义在Object类中，根据对象在内存中的地址来计算哈希值，当然，我们也可以在Object的子类中重写hashCode()函数，常用来作为键的Integer、String类的hashCode()函数如下所示。从代码中，我们可以发现，Integer对象的哈希值就是其所表示的数值（value）本身，例如，Integer(11)的哈希值为11，Integer(334)的哈希值为334。
~~~java
// Integer类的hashCode()函数
public int hashCode() {
    return value;
}

// String类的hashCode()函数
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
~~~

### 在hash()函数中，为什么不直接使用key的hashCode()函数返回值作为哈希值呢？
一般来说，table数组的大小n不会很大，一般会小于2^16（65536）。而hashCode()函数的返回值h为int类型，长度为4个字节。在计算key对应的数组下标时，h跟n求模后，h的高16位信息将会丢失，相当于只使用了h的后16位信息。理论上讲，参与计算的信息越多，得到的数组下标越随机，数据在哈希表的各个链表中的分布就会越均匀。因此，我们将hashCode()函数的返回值与其高16位异或，这样所有的信息都没有浪费。

那么，这里为什么使用异或而非与、或来对h和h>>>16进行操作呢？
这是因为异或产生的结果更加随机。如下图所示，二进制位进行与操作，0出现的概率更高，二进制位进行或操作，1出现的概率更高，而只有二进制位进行异或操作，1和0出现的概率才相等。

### 为何HashMap中数组的大小必须是2的幂次方？
2的幂次方将其减一之后的二进制串跟其他数进行位运算求与，就相当于对这个数进行取模求余操作，即方便位运算快速计算出存储在数组的下标

### key可为null值
从上述hash()函数的代码实现中，可以发现，值为null的key的哈希值为0，对应数组下标为0，也就是说，值为null的key也可以存储在HashMap中。不过，一个HashMap容器只能存储一个值为null的key，这符合HashMap容器不允许存储重复key的要求。

## 键的不可变性
Node类中的hash属性。hash属性存储的是key的哈希值（也就是通过hash(key)计算得到的值）。这个值的作用是预判等，提高查询速度。将这个值作为属性存储在节点中目的是避免重复计算。

当我们调用get(xkey)函数查询键xkey对应的值（value）时，HashMap容器先通过hash(xkey)函数计算得到xkey的哈希值，假设为xhash。xhash跟table数组的大小n取模，假设得到的数组下标为xIndex，也就说明x应该出现在table[xIndex]对应的链表中。

遍历table[xIndex]所对应的链表，查找属性key等于xkey的节点。当遍历到某个节点node之后,首先会拿node.hash，与x的哈希值xhash比较，如果不相等，则说明node.key跟xkey肯定不相等，就可以继续比较下一节点了。

如果node.hash跟xhash相等，那么，这也并不能说明这个节点就是我们要找的节点。因为哈希函数存在一定的冲突概率，所以，即便哈希值相等，node.key也未必就跟xkey相等。因此，我们需要再调用equals()方法，比较node.key与xkey是否真的相等，如果相等，那么这个节点就是我们要查找的节点。如果不相等，则继续遍历下一个节点，然后再进行上述比较。

node.key和xkey为对象，需要调用equals()函数比较是否相等。node.hash和xhash为整数，直接使用等号即可判等。后者比前者的执行效率更高。通过预先比对哈希值，过滤掉node.key和xkey不可能相等的节点，以此来提高查询速度。

上述查找过程对应到HashMap类中的源码如下所示。
~~~java
//HashMap类的源码
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && ((k = first.key) == key 
            || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            //...省略部分代码...
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || 
                    (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
~~~

一般来讲，我们使用Integer、Long、String等类型的数据作为键。这些类型有一个共同的特点，那就是不可变。如果键在存入HashMap之后被更改，而其在table数组中存储的下标不随之变化的话，那么，这个键将无法再被查询到。

尽管在绝大部分情况下，我们都会选择使用Integer、Long、String等Java内置不可变类作为HashMap容器的键，但是，在极个别情况下，我们也有可能需要使用自定义类作为键，如下代码中的SequenceId类。对于作为键的自定义类，我们需要重写Object父类的两个函数：一个是用于计算哈希值的hashCode()函数，另一个是用于数据判等的equals()函数。除此之外，为了避免键被更改之后无法再被查询到，自定义类中参与实现hashCode()函数和equals()函数的属性需要设置为不可变的。
~~~java
public class SequenceId {
    private int zoneId;
    private int machineId;
    private String id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || this.getClass() != o.getClass()) {
            return false;
        }
        SequenceId that = (SequenceId) o;
        if (this.zoneId == that.zoneId && this.machineId == that.machineId
                && this.id.equals(that.id)) {
            return true;
        }
        return false;
    }

    @Override
    public int hashCode() {
        return Objects.hash(this.zoneId, this.machineId, this.id);
    }
}
~~~

## 装载因子
为了方便使用，JCF提供的容器基本上都支持动态扩容。当容器容纳不下新的元素时，容器便会开启扩容，将数据搬移到更大的存储空间中。HashMap使用链表法解决哈希冲突，并且链表可以无限长，因此，HashMap不存在无法容纳新元素的情况。但是，当HashMap容器中的数据越来越多时，在table数组大小不变的情况下，链表的平均长度会越来越长，进而影响到HashMap容器中各个操作的执行效率，此时就需要扩容了。

具体什么时候扩容，主要由table数组的大小（n）和装载因子（loadFactor）决定。当HashMap容器中的元素个数超过n * loadFactor时，就会触发扩容。其中，n * loadFactor在HashMap类中定义为属性threshold。

在HashMap中，装载因子laodFactor的默认值为0.75，table数组的默认初始大小为16。也就说，当添加元素个数超过12（16*0.75）个时，HashMap容器就会触发第一次扩容。当然，我们也可以通过带参构造函数，自定义table数组的初始大小和装载因子，如下代码所示。
~~~java
//HashMap类的其中一个构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException(
           "Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException(
           "Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
~~~

前面讲到，为了便于使用位运算来实现取模运算，table数组的大小必须是2的幂次方，但是，如果通过构造函数，传入的参数initialCapacity不是2的幂次方，那么，又该怎么办呢？实际上，在上述代码中，使用tableSizeFor()函数就是为了解决这个问题。tableSizeFor()函数的代码实现如下所示。tableSizeFor()函数会寻找比initialCapacity大的第一个2的幂次方数。比如，tableSizeFor(7)=8，tableSizeFor(13)=16。
~~~java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) 
                   ? MAXIMUM_CAPACITY : n + 1;
}
~~~

这里稍微解释一下，以免细心的你会有疑问，tableSizeFor(initialCapacity)的值赋值给了threshold，这似乎有些不对。因为tableSizeFor(initialCapacity)的值是table数组的大小，threshold是触发扩容的阈值，按理来说，tableSizeFor(initialCapacity)的值乘以loadFactor才应该是threshold。实际上，这是因为在创建HashMap对象时，table数组只声明未创建，其值为null。只有当第一次调用put()函数时，table数组才会被创建。但是，HashMap并没有定义表示table数组大小的属性，于是，tableSizeFor(initialCapacity)的值就暂存在了threshold属性中，当真正要创建table数组时，HashMap会使用如下所示代码，先使用threshold作为数组大小创建table数组，再将其重新赋值为真正的扩容阈值。
~~~java
this.table = new T[this.threshold];
this.threshold *= this.factor;
~~~

装载因子的默认值0.75是权衡空间效率和时间效率，精心挑选出来的。在平时的开发中，我们不要轻易修改装载因子，除非我们对时间和空间有比较特殊的要求，比如，如果更关注时间效率，我们可以适当减小装载因子，这样哈希冲突的概率会更小，链表长度更短，增删改查操作更快，但空间消耗会更大；相反，如果更关注空间效率，我们可以适当增大装载因子，甚至可以将其设置为大于1，这样table数组中的空闲空间就更少，不过，这也会导致哈希冲突概率更大，链表长度更长，增删改查以及扩容都会变慢。

## 动态扩容
当调用put()函数往HashMap容器中添加键值对之后，如果HashMap容器中键值对的个数是否超过threshold（table数组大小*装载因子），那么就会触发HashMap的动态扩容，即申请一个新的table[]数组，大小为原table数组的2倍，并将原table数组中的链表节点，一个一个的搬移到新的table[]数组中。

在进行扩容时，HashMap会逐一处理table数组中的每条链表。在JDK8中，HashMap中的table数组还有可能存储的是红黑树而非链表（这点待会会讲）。因为红黑树的处理方法，跟链表的处理方法类似，所以，我们拿链表举例讲解。

因为新的table数组大小newCap是原table数组大小oldCap的两倍，所以，一些节点在新table数组中的存储位置将会改变，我们需要重新计算其对应的数组下标。但因为每个节点的key的哈希值，已经存储在节点的hash属性中，所以，不需要调用哈希函数重新计算，只需要将节点node中存储的hash值跟newCap取模即可。取模操作仍然可以使用位运算来替代，也就是node.hash & (newCap-1)，由此得到节点node搬移到新的table数组中的位置下标。

实际上，在JDK8中，新的位置下标并非通过node.hash跟newCap取模计算得到的，其计算过程做了更进一步的优化：如果node.hash & oldCap == 0，则节点在新table数组中的下标不变；如果node.hash & oldCap != 0，则节点在新table数组中的下标变为i+oldCap（i为节点在原table数组中的下标）。

HashMap依次扫描table数组中的每一条链表，根据节点在新table数组中的下标是否更改，将链表中的节点分配到lo链表或hi链表。lo链表中存储的是下标值未变的节点（节点在table数组中的位置下标为i，在新的table数组中的位置下标也是i），hi链表中存储的是下标值有所改变的节点（节点在新的table数组中的位置下标变为j）。扫描完一条链表之后，HashMap将lo链表存储到新的table数组中下标为i的位置，将hi链表存储到新的table数组中下标为j的位置。

## 链表树化
尽管我们可以通过装载因子，使HashMap容器中不会装载太多的键值对，但这只能限制平均链表长度，无法限制单个链表的长度。如下代码所示，12个键值对均存储在table数组中下标为1的链表中。虽然链表的平均长度为1，但是，最大链表长度为12。
~~~java
Map<Integer, String> map = new HashMap<>(); //默认table大小为16
for (int i = 0; i < 12; ++i) { //16*0.75=12，超过12个键值对才扩容
  map.put(1+i*16, "value"+i);
}
~~~

链表过长会导致HashMap性能下降，针对这个问题，JDK8做了一些优化。当链表中的节点个数大于等于8，并且table数组的大小大于等于64时，HashMap会将链表转化为红黑树。假设原链表中包含n个节点，那么，增删改查操作的时间复杂度为O(n)。当我们将链表转化为红黑树之后，增删改查操作的时间复杂度变为O(logn)，性能大大提高。我们把将链表转化为红黑树的过程叫做链表树化（treeify）。

不过，链表树化比较耗时，并且，存储同样个数的节点，红黑树占用的空间要比链表更大（红黑树节点包含两个指针，而链表节点只包含一个指针），因此，我们希望链表树化极少发生。如果table数组长度小于64，即便链表中的节点个数大于等于8，那么，这种情况也不会触发链表树化，而是会触发动态扩容。HashMap试图通过扩容将长链表拆分为短链表。在小数据量的情况下，扩容要比树化更简单、更省时间。

跟树化相反的过程叫做反树化（untreeify）。当红黑树中节点个数较少时，HashMap会将红黑树重新转换回链表。毕竟维护红黑树平衡的成本比较高，对于少数节点来说，使用链表存储比使用红黑树存储更加高效。触发反树化的场景有两个：一个是删除操作，另一个是动态扩容。

我们先来看删除操作。当删除操作执行完成之后，HashMap会检查红黑树是否需要进行反树化。不过，是否需要进行反树化，并非直接由红黑树中的节点个数来决定，而是由红黑树的树形结构来决定，如下代码所示。
~~~java
if (root == null || (movable && (root.right == null
                                 || (rl = root.left) == null
                                 || rl.left == null))) {
    tab[index] = first.untreeify(map);  // too small
    return;
}
~~~

通过上述代码，我们反推得到这样的结论：触发反树化的红黑树的节点个数处于[2,6]之间。红黑树结构的不同，对应触发反树化的节点个数也不同，但当红黑树中节点个数超过6个时，肯定不会触发反树化。也就是说，尽管树化的阈值是8，但是反树化的阈值却不是8，而是[2,6]之间的某个数。之所以树化的阈值和反树化的阈值不相等，是为了避免频繁的插入删除操作，导致节点个数在7、8之间频繁波动，进而导致链表和红黑树之间频繁的转换。毕竟转换操作也是耗时的。

我们再来看动态扩容。前面讲到，在进行动态扩容时，每一条链表都会分割为lo和hi两条链表，同理，每一个红黑树也会分割为lt和ht两个红黑树。lt中存储的是下标位置不变的节点，ht中存储的是下标位置变化的节点。不过，在构建lt和ht之前，HashMap会先统计下标位置不变的节点个数lc，以及下标位置变化的的节点个数hc。如果lc小于等于6，在新的table数组中，HashMap会使用链表来存储下标不变的节点。同理，如果hc小于等于6，在新的table数组中，HashMap会使用链表来存储下标改变的节点。


