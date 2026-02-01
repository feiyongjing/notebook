## CAS介绍
在synchronized和Lock的实现原理中反复提到CAS。比如，多个线程竞争锁时，通过CAS来判定谁获得锁。多个线程同时操作AQS中的等待队列时，使用CAS保证操作的线程安全性。除此之外，后面讲到的原子类、并发容器，其底层实现也会用到CAS。CAS是多线程开发基础中的基础。

CAS指的是先检查后更新这类复合操作，英文翻译有多种：Compare And Set、Compare And Swap或Check And Set。先来看下面这样一段代码。这段代码是从Lock的底层实现原理中，抽象出来的一段功能代码。其中，state为共享变量，值为0表示没有加锁，值为1表示已加锁。多个线程同时调用tryAcquire()函数，谁将state变为1，谁就获取到了锁。
~~~java
public class LockDemo {
  private int state = 0;
  public boolean tryAcquire() {
    if (state == 0) {
      state = 1;
      return true;
    }
    return false;
  }
}
~~~

tryAcquire()函数中的代码访问共享资源，并且包含复合操作，符合临界区的特征。多个线程竞态交叉执行tryAcquire()函数，有可能存在这样的情况：多个线程均检测到state等于0，并先后将其值设置为1，导致多个线程同时获取到同一把锁。因此，tryAcquire()是非线程安全的。

为了保证线程安全，可以使用synchronized或Lock对tryAcquire()函数加锁。不过，如果上述代码是synchronized或Lock的底层实现中的一部分，那么，就不能在实现锁（synchronized或Lock）时再递归地使用锁。可以使用硬件层面提供的CAS指令来保证tryAcquire()函数的原子性。

## CAS硬件指令
X86提供的CAS指令为cmpxchg指令。指令格式如下所示。cmpxchg指令涉及三个操作数。目标操作数位于寄存器或内存中，用于存储变量的当前值C（CurrentValue）。源操作数位于寄存器中，用于存储变量的更新值N（NewValue）。隐藏的操作数位于AX寄存器中，在指令中没有明确指出，用于存储变量的期望值E（ExpectedValue）。
~~~
cmpxchg [目标操作数]，[源操作数]
~~~

在执行cmpxchg指令时，CPU会判断变量的当前值C是否等于期望值E。如果当前值C跟期望值E相等的话，那么，CPU将更新值N赋值给存储当前值C的寄存器或内存中，并设置标志寄存器中的ZF标志位为1，以表示变量值更新成功。如果当前值C跟期望值E不相等，那么，CPU将当前值C赋值给存储期望值E的寄存器中，也就是AX寄存器中，并设置标志寄存器中的ZF标志位为0，以表示变量值更新失败。

在单核计算机上，cmpxchg指令是原子操作。尽管cmpxchg指令包含很多细分操作，看似是非原子的复合操作，但是，指令是CPU执行的最小单元，指令执行的过程不可中断。当多个线程共用一个CPU来交替执行指令时，只有当前线程执行完正在执行的指令（比如cmpxchg指令）之后，操作系统才可以调度CPU执行其他线程，其他线程是看不到cmpxchg指令执行的中间状态的。因此，cmxchg在单核计算机上是原子操作。

不过，在多核计算机上，cmpxchg指令是非原子操作。在多核计算机上，多个线程可以并行运行在多个CPU上，也就是说，多个线程可以并行执行cmpxchg指令，同时对同一个内存变量进行CAS操作，因此，cmpxchg就不再是原子操作了。在多核计算机中，为了保证cmpxchg指令的原子性，需要在cmpxchg指令前加LOCK前缀，如下所示。
~~~
LOCK cmpxchg [目标操作数], [源操作数]
~~~

在讲解volatile如何解决内存可见性问题时，也提到过LOCK前缀，当时，LOCK前缀的作用是同步缓存。实际上，LOCK前缀不仅仅可以同步缓存，还可以锁定总线，禁止多个CPU同时操作一块共享的内存单元。这样就能保证在多核计算机上，同一时刻只有一个CPU在执行cmpxchg指令。实际上，这就相当于在硬件层面给cmpxchg指令加了锁。

## CAS的native方法
JVM的Unsafe类中提供了大量的native方法，对比较底层的操作进行了封装，比如之前讲到的用于阻塞和唤醒线程的park()和unpark()方法就是native方法。Unsafe类中提供了3个封装了CAS指令的native方法，如下代码所示。其中，o表示针对哪个对象的成员变量进行CAS操作。offset表示成员变量在对象中的偏移位置。oldValue为期望值，newValue为更新值。如果对象o中偏移位置为offset的成员变量的值等于oldValue，那么，这个成员变量的值将被更新为newValue，并且CAS方法返回true。
~~~java
public final native boolean compareAndSwapInt(
        Object o, long offset, int oldValue, int newValue);
                                       
public final boolean compareAndSwapLong(
        Object o, long offset, long oldValue, long newValue);

public final boolean compareAndSwapObject(
        Object o, long offset, Object oldValue, Object newValue);
~~~

以上3个CAS方法的代码实现类似，拿其中的compareAndSwapInt()方法举例讲解。Java是跨平台语言。针对不同的平台，compareAndSwapInt()方法的代码实现不同。在Linux_X86平台（CPU为X86，操作系统为Linux）下，compareAndSwapInt()方法的代码实现如下所示。在Hotspot JVM中，native方法由C++代码实现，因此，需要在JVM源码中查看native方法的代码实现。
~~~c++
//以下代码位于unsafe.cpp中
UNSAFE_ENTRY(jboolean, Unsafe_CompareAndSwapInt(
  JNIEnv *env, jobject unsafe, jobject obj, jlong offset, jint e, jint x))
  UnsafeWrapper("Unsafe_CompareAndSwapInt");
  oop p = JNIHandles::resolve(obj);
  jint* addr = (jint *) index_oop_from_field_offset_long(p, offset);
  return (jint)(Atomic::cmpxchg(x, addr, e)) == e;
UNSAFE_END
~~~

上述代码又调用了Atomic类中的cmpxchg()方法。cmpxchg()方法的代码实现如下所示。cmpxchg()方法通过在C++代码中调用汇编代码来执行cmpxchg指令。
~~~c++
inline jint     Atomic::cmpxchg(
  jint exchange_value, volatile jint* dest, jint compare_value) {
  int mp = os::is_MP();
  __asm__ volatile (LOCK_IF_MP(%4) "cmpxchgl %1,(%3)"
                    : "=a" (exchange_value)
                    : "r" (exchange_value), "a" (compare_value), "r" (dest), "r" (mp)
                    : "cc", "memory");
  return exchange_value;
}
~~~

了解了Unsafe类中CAS方法的实现原理之后，使用这些CAS方法来实现tryAcquire()函数，对应的代码实现如下所示。代码比较简单，就不详细解释了。
~~~java
public class Demo {
  private int state = 0;

  private static final Unsafe unsafe = Unsafe.getUnsafe();
  private static final long stateOffset;
  static {
    try {
      stateOffset = unsafe.objectFieldOffset(Demo.class.getDeclaredField("state"));
    } catch (Exception ex) { throw new Error(ex); }
  }
  
  public boolean tryAcquire() {
    return unsafe.compareAndSwapInt(this, stateOffset, 0, 1);
  }
}
~~~

## CAS执行失败处理
如果多个线程竞争执行CAS，那么，只有一个线程会执行成功，其他执行失败的线程又该如何处理呢？实际上，根据不同的业务场景，可以选择不同的处理策略，既可以转去执行失败处理逻辑，也可以自旋执行CAS，直到执行成功为止。

举例解释一下自旋执行CAS这种处理策略，示例如下代码所示。如果希望increment()函数线程安全，那么，现在有两种处理方法，一种是使用synchronized或Lock对increment()函数加锁，对应代码为increment_lock()函数，另一种是使用CAS，对应代码为increment_CAS()函数。
~~~java
public class Demo {
  private int id = 0;
  
  // 原始方法
  public void increment() {
    id++;
  }

  // 线程安全处理方法一：使用锁
  public void increment_lock() {
    synchronized (this) {
      id++;
    }
  }

  // 线程安全处理方法二：使用CAS
  private static final Unsafe unsafe = Unsafe.getUnsafe();
  private static final long idOffset;
  static {
    try {
      idOffset = unsafe.objectFieldOffset(
                  Demo.class.getDeclaredField("id"));
    } catch (Exception ex) { throw new Error(ex); }
  }

  public void increment_CAS() {
    int oldValue = id;
    int newValue = oldValue+1;
    unsafe.compareAndSwapInt(this, idOffset, oldValue, newValue);
  }
}
~~~

对比以上两种处理方法，increment_lock()函数总是能让id值增一，但increment_cas()却不能，在CAS失败时，函数直接返回，id值并没有增一。也就是说，increment_cas()的处理方式，并不符合对increment()函数的逻辑要求（即增一）。对于这个问题，就可以使用自旋+CAS来解决。如下代码所示。
~~~java
public void increment_CAS() {
  boolean succeded = false;
  while (!succeded) {
    int oldValue = id;
    int newValue = oldValue + 1;
    succeded = unsafe.compareAndSwapInt(this, idOffset, oldValue, newValue);
  }
}
~~~

## 悲观锁和乐观锁
前面讲过很多锁，比如偏向锁、轻量级锁、自旋锁等等，这里再介绍另外两种锁：悲观锁和乐观锁，它们属于抽象的概念，并不是具体实现。乐观锁指的是乐观的认为代码不大可能存在资源竞争，大部分情况都不需要加锁。悲观锁指的是悲观的认为代码很有可能会出现资源竞争，绝大部分情况都需要加锁。synchronized或Lock可以用来实现悲观锁，自旋+CAS可以用来实现乐观锁。

### 悲观锁和乐观锁的利弊
- 基于synchornized或Lock实现的悲观锁，等待资源而阻塞线程会导致内核态到用户态的上下文切换，带来性能损耗，但是，处于阻塞状态的线程不会被分配CPU时间片，不会浪费CPU资源。
- 基于自旋+CAS实现的乐观锁，循环执行CAS，不需要阻塞线程，没有内核态到用户态的上下文切换带来的性能损耗，但是，线程一直处于运行状态，白白浪费CPU资源。
- 如果多线程竞争资源不激烈，那么，使用乐观锁来竞争资源更合适，如果多线程竞争资源比较激烈，那么，使用悲观锁来竞争资源更合适。

## 应用场景
### 如何在不使用锁的情况下实现AQS中的线程安全的等待队列?
等待队列主要包含两个操作。一个是锁竞争失败之后线程入队，另一个是等待队列中的线程获取到锁之后出队。等待队列是基于双向链表来实现的，非线程安全的等待队列的代码实现如下所示。其中，引入虚拟头节点和tail尾指针是为了方便在链表尾部插入元素。
~~~java
public class AQSDemo {
  public static final class Node {
    private int threadId;
    private Node prev;
    private Node next;
    public Node(int val, Node prev, Node next) {
      this.threadId = val;
      this.prev = prev;
      this.next = next;
    }
  }
  private Node head = new Node(-1, null, null); //虚拟头节点
  private Node tail = head;

  public void addWaiter(int threadId) { //入队
    Node newNode = new Node(threadId, null, null);
    newNode.prev = tail;
    tail.next = newNode;
    tail = newNode;
  }

  public void removeWaiter() { //出队
    if (head.next == null) return;
    head = head.next;
    head.threadId = -1;
    head.prev = null;
  }
}
~~~

上述代码中的addWaiter()函数访问共享资源（tail），并且包含复合操作（读写tail和tail.next），多线程竞态交叉执行addWaiter()函数会存在线程安全问题
可以使用CAS来保证addWaiter()函数的线程安全性，如下代码所示。多个线程竞争往链表尾部添加元素时，只有一个线程会成功执行CAS，将tail指针更新为指向自己的节点，其他线程执行CAS失败，继续自旋执行CAS，直到将元素成功添加到链表尾部为止。
~~~java
public void addWaiter(int threadId) {
  Node newNode = new Node(threadId, null, null);
  for(;;) {
    Node oldTail = tail;
    if (unsafe.compareAndSwapObject(this, tailOffset, oldTail, newNode)) {
      newNode.prev = oldTail;
      oldTail.next = newNode;
      return;
    }
  }
}
~~~

通过使用CAS重构addWaiter()入队函数，使得入队与入队之间线程安全，那么，在不对出队函数removeWaiter()进行重构的情况下，入队与出队之间是否线程安全呢？
答案是安全。这得益于虚拟头节点的存在以及通过替代来删除真实头节点的处理思路。出队将虚拟头节点删除，然后将真实头节点设置为虚拟头节点。同时执行入队和出队，出队操作只会影响X节点的prev指针和值，入队只会影响X节点的next指针。因此，入队和出队操作互相不影响。

入队和入队之间、入队和出队之间均是线程安全的，那么，出队和出队之间是否线程安全呢？
答案是不安全。参照removeWaiter()的代码实现，两个线程同时执行removeWaiter()函数，有可能删除的是同一个真实头节点。为了解决出队与出队之间的线程安全问题，可以使用类似addWaiter()的重构方式来重构removeWaiter()，代码如下所示。
~~~java
  public void removeWaiter() {
    for (;;) {
      Node oldHead = head;
      if (oldHead.next == null) return;
      if (unsafe.compareAndSwapObject(this, headOffset,oldHead,oldHead.next)) {
        head.threadId = -1;
        head.prev = null;
        return;
      }
    }
  }
~~~
不过，对于AQS来说，等待队列实际上并不需要保证出队与出队之间的线程安全性，这是因为对于独享锁来说，一次只会有一个线程竞争到锁，不会出现多个线程同时执行出队的情况。当然对于读写锁的读锁（共享锁）还是需要进行CAS+自旋操作

### 使用CAS或者自旋+CAS就是无锁编程，这样的说法严谨吗？
不合理，CAS使用了底层的总线锁



