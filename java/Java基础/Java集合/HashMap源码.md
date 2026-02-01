

~~~java
/**
 * @param hash 是key的hash值
 * @param key 
 * @param value
 * @param onlyIfAbsent 如果为true，则不会使用新的value更改原来的value
 * @param evict 如果为false，则表处于创建模式
 * @return 原来的值，如果没有则为空
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        //tab是哈希桶数组、p是其中一个哈希桶、n是哈希桶数组长度、i是key计算后分配的哈希桶数组索引。这些变量后续会一一赋值
        Node<K,V>[] tab; Node<K,V> p; int n, i; 
        
        // 判断现有哈希桶数组是否是空的，是则对哈希桶数组扩容。变量tab和n赋值
        if ((tab = table) == null || (n = tab.length) == 0) 
            n = (tab = resize()).length;  
            
        // 计算key分配的哈希桶数组索引并获取对应的哈希桶分别赋值给i和p，判断对应的哈希桶是否不存在，若不存在就创建哈希桶
        if ((p = tab[i = (n - 1) & hash]) == null) 
            tab[i] = newNode(hash, key, value, null);
        else {
            // e是即将存储key和value的哈希桶中链表或红黑树节点，k是哈希桶链表或红黑树节点的key。这些变量后续会一一赋值
            Node<K,V> e; K k; 
            
            // 通过哈希值和equals来判断哈希桶链表的第一个节点是否是key和value的存储节点，若是则将第一个节点赋值给变量e
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode) // 判断哈希桶内部是否是红黑树，若是则找到存储对应key和value的红黑树节点赋值给变量e
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                // 遍历哈希桶链表，直到找到节点存储key和value，若找不到就在哈希桶链表中添加一个节点存储key和value
                for (int binCount = 0; ; ++binCount) {
                    // 先将哈希桶链表的下一个节点赋值给变量e，判断哈希桶链表是否到尽头了，到尽头了就在尽头创建新的链表节点存储key和value
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        // 判断哈希桶链表长度是否大于阈值，若大于则将链表转化成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 通过哈希值和equals来判断哈希桶链表的下一个节点是否是key和value的存储节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 哈希桶链表节点向后移动
                    p = e;
                }
            }
            if (e != null) { // 判断哈希桶链表节点是否有相同的key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null) // 判断是否覆盖旧的value
                    e.value = value;
                afterNodeAccess(e);
                return oldValue; // 返回旧值
            }
        }
        ++modCount;
        if (++size > threshold) // 判断容量大小是否超过总容量乘以负载因子，超过就扩容
            resize();
        afterNodeInsertion(evict);
        return null;
    }


~~~