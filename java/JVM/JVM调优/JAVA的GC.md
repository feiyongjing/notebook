
# 内存管理的两个流派
  - ⼿动内存管理（C/C++）
  - ⾃动内存管理（⼏乎所有的现代编程语⾔）
    - 引⽤计数（Reference Counting）：一个对象没有被引用则回收。引⽤计数的致命缺点：对象相互循环引用引用
    - 标记清除（Mark and Sweep）：标记一组根节点（GC Roots）。GC Roots可以到达的对象不是垃圾其他的都是垃圾

# 哪些东⻄是GC Roots？
  - 活的线程(java.lang.Thread)
  - 类的静态成员
  - 线程⽅法栈所引⽤的对象
  - JNI引⽤的对象
  - 分代GC时其他代的对象
---

# 内存的碎⽚化问题
  - ⻓期⼯作的内存会出现碎⽚化
  - 总可⽤空间仍然⾜够，但是⽆法分配⼤对象
  - 垃圾回收=回收可⽤空间+压缩内存 

# 引⽤的类型
  - 强引⽤ Strong Reference：你⽤到的都是强引⽤,  Object obj = new Object(); obj是一个指向Object对象的强引用。只要obj存在，Object对象就不会被垃圾回收器回收
  - 软引⽤ Soft Reference：内存不够的时候会回收他们 Object obj = new Object(); SoftReference<Object> softRef = new SoftReference<>(obj); softRef是一个指向Object对象的软引用。当内存不足时，垃圾回收器会回收Object对象，同时将softRef指向null。
  - 弱引⽤ Weak Reference：⼀GC就回收它们 Object obj = new Object(); WeakReference<Object> weakRef = new WeakReference<>(obj); 当下一次垃圾回收时，垃圾回收器会回收Object对象，同时将weakRef指向null
  - 影⼦引⽤(虚引用) Phantom Reference
    - 影子引用的get()方法永远返回null，因此无法通过影子引用来访问对象。影子引用的主要作用是在对象被垃圾回收器回收时，可以收到一个系统通知，从而进行一些资源清理工作。影子引用通常用于管理直接内存（Direct Memory）或者与外部资源（例如文件或者网络连接）关联的对象。在这些情况下，对象的生命周期不受Java虚拟机的管理，因此需要在对象被回收时进行一些清理工作，例如释放直接内存或关闭文件句柄。通过使用影子引用，可以在对象被回收时得到一个通知，从而进行这些清理工作。

## 影子引用例子代码
~~~java
import java.lang.ref.*;

public class PhantomReferenceExample {
    public static void main(String[] args) {
        // 创建一个需要管理的对象
        Object obj = new Object();

        // 创建一个ReferenceQueue对象
        ReferenceQueue<Object> queue = new ReferenceQueue<>();

        // 创建一个PhantomReference对象，并将其与需要管理的对象关联
        PhantomReference<Object> ref = new PhantomReference<>(obj, queue);

        // 将obj赋值为null，让它成为垃圾对象
        obj = null;

        // 不断循环，直到obj被回收
        while (true) {
            // 从ReferenceQueue中获取被回收的PhantomReference对象
            Reference<?> r = queue.poll();
            if (r != null) {
                // 执行资源清理工作
                System.out.println("Cleaning up...");
                // ...
                break;
            }
            // 等待一段时间，让垃圾回收器有机会回收obj
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~
---

# GC发⽣了什么？
  - Stop the world（STW）
  - 清除垃圾
  - 压紧内存

# 对象的分代
## 内存分代
- 年轻代（Young Generation）
- ⽼年代（Old Generation/Tenured）
- 永久代（Permanent Generation）
 
### 年轻代
- Eden（伊甸园）
  可以分为多个Thread Local Allocation Buffer 
  Survivor（S0/S1或者from/to）：永远都有一块空白的Survivor区
  Young GC发⽣时，整个年轻代和Survivor区中不够⽼的对象进⼊空白的Survivor区，而Survivor区⾜够⽼的对象被提升⾄⽼年代（Tenured），留下一个空白的Survivor区  
- ⽼年代
  ⽼年代相对较⼤
  存储⾜够年⽼的对象
  发⽣GC的频率较低
- 永久代/元空间
  Java 8之前：永久代是堆的⼀部分，存储类数据、字符串常量
  Java 8+：元空间，不是堆的⼀部分（-Xmx）除⾮特殊指定，否则没有上限。设置是 -XX:MaxMetaspaceSize/OOM: metaspace 

# GC的种类
  Minor GC/Young GC
  - 总是发⽣在Eden满的时候
  - 会发⽣STW
  Major GC vs Full GC
  - Major GC清除⽼年代
  - Full GC清除整个堆 


# 垃圾回收的过程 
 -XX:+PrintGCDetails
 -XX:+PrintGCDateStamps
 -XX:+PrintGCTimeStamps 