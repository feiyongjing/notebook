### 相对Java synchronized，JUC Lock有什么优势？
JUC并发包提供的Lock锁。相对于synchronized锁，Lock锁提供了更加丰富的特性，比如支持公平锁、可中断锁、非阻塞锁、可超时锁等。

## 锁的概述
JUC提供了几种不同的锁，继承和实现层次关系如下
- Lock接口及其实现类ReentrantLock
- 读写锁ReadWriteLock接口及其实现类ReentrantReadWriteLock
- 读写锁的升级版StampedLock

先来看Lock接口。接口定义如下所示。因为在平时的开发中，我们用到的锁都是可重入锁，所以，Lock接口只有 一个可重入的实现类ReentrantLock。
~~~java
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;   // 可中断锁，用于阻塞获取锁，如果阻塞期间线程被终止，就抛出异常，一般用于线程终止时抓取异常处理资源的关闭
    boolean tryLock();                                      // 非阻塞锁，尝试获取锁，不阻塞立即返回结果
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;  // 可超时锁，获取不到锁就阻塞等待一段时间，超过了设定的超时时间，线程仍然没有获取到锁就直接返回，并且是可被中断的
    void unlock();
    Condition newCondition();
}
~~~

### 可重入锁
可重入锁指的是可以被同一个线程多次加锁的锁。注意，这里说的多次加锁，并不是说解锁之后再次加锁，而是在锁没有解锁前再次加锁。如下代码所示，为了保证线程安全，getEvenSeq()函数和increment()函数中的代码都加了锁。getEvenSeq()函数调用increment()函数，导致getEvenSeq()函数在锁释放前再次加锁。
如果JUC提供的锁不支持可重入特性，那么，getEvenSeq()中的第二次加锁需要等待锁释放，而锁释放又需要increment()函数加锁之后才能执行，于是，getEvenSeq()就会出现死锁。
~~~java
public class Demo {
  Lock lock = new ReentrantLock();
  private int seq = 0;

  public int getEvenSeq() {
    lock.lock();
    try {
      // ...省略其他操作...
      if (seq%2 == 1) {
        increment();
      }
      return seq;
    } finally {
      lock.unlock();
    }
  }

  public void increment() {
    lock.lock();
    seq++;
    lock.unlock();
  }
}
~~~

对于上述代码，我们再稍微解释一下。之所以getEvenSeq()函数使用finally来释放锁，是为了避免代码抛出异常而导致锁无法正常释放。而之所以increment()函数没有使用finally来释放锁，是因为代码比较简单，我们可以确定代码不会抛出异常。

JUC提供的锁都是可重入锁。实际上，Java synchronized内置锁也是可重入锁。从侧面上，我们也可以得出，可重入是对锁的基本要求。为了实现可重入特性，可重入锁中需要有一个变量来记录重入的次数。每重入一次，变量就增一；每解锁一次（调用unlock()或退出synchronized代码块），变量就减一，直到变量值为0时，才会释放锁并唤醒其他线程。

### 公平锁
对于公平锁来说，线程会按照请求锁的先后顺序来获得锁，也就是我们经常说的FIFO。对于非公平锁来说，当多个线程请求锁时，非公平锁无法保证这些线程获取锁的先后顺序，有可能后申请锁的线程先获取到锁。

Java将synchronized设计为只支持非公平锁，而JUC提供的ReentrantLock既支持公平锁，也支持非公平锁。默认情况下，ReentrantLock为非公平锁。如果需要创建公平锁，那么，我们只需要在创建ReentrantLock对象时，将构造函数的参数设置为true即可。如下代码所示。
~~~java
Lock lock = new ReentrantLock(true); // 公平锁
~~~

接下来看下，公平锁和非公平锁的实现原理。

在讲解synchronized的底层实现原理时讲到，当多个线程竞争锁时，竞争到锁的线程就去执行任务了，没有竞争到锁的线程会放入Monitor锁的_cxq队列中等待锁，并且还需要调用park()函数阻塞自己。当持有锁的线程释放锁时，它会从_EntryList队列中取一个线程，取消阻塞状态，让它去重新竞争锁，而不是直接将锁给它。如果此时有新来的线程也要竞争这个锁，新来的线程有可能不需要排队，就能直接获取到锁，显然，这是一种“插队”行为。因此，synchronized是非公平锁。

当然，我们也可以让synchronzied支持公平锁。实现起来也很简单。新来的线程在申请获取锁时，如果_EntryList队列和_cxq队列中有排队等待锁的线程，那么，不管此时锁有没有释放，新来的线程都直接放入_cxq队列中排队，按照先来后到的顺序等待锁，以避免新来的线程的“插队”行为。这样实现的锁就是公平锁。

实际上，ReentrantLock实现公平锁和非公平锁的方法，跟上述synchronized的实现方法，其基本原理是一致的。区别在于，ReentrantLock使用AQS（抽象队列同步器）来存储排队等待锁的线程。关于AQS，在下面进行详细讲解。

既然实现公平锁并不复杂，而且从直觉上，公平锁比非公平锁更加合理，那么，synchronized为什么只支持非公平锁？主要原因有以下3个方面。
- 历史的原因：synchronized早期开发时并没有考虑那么全面；
- 需求的原因：绝大部分业务场景都不需要严格规定线程的执行顺序，如果真的需要，我们可以通过条件变量（wait()、notify()等）等同步工具来实现；
- 性能的原因：非公平锁的性能比公平锁的性能更好。我们知道，加入等待队列并调用park()函数阻塞线程，涉及到用户态和内核态的切换，这个过程是比较耗时。对于非公平锁来说，新来的线程直接竞争锁，这样就有可能避免加入等待队列并调用费时的park()函数。

不过，非公平锁也有缺点，在锁竞争比较激烈的情况下，频繁有新来的线程插队，就有可能导致排在等待队列中的线程迟迟无法获取到锁。

### 可中断锁
对于synchronized锁来说，线程在阻塞等待synchronized锁时是无法响应中断的。而JUC Lock接口提供了lockInterruptibly()函数，支持可响应中断的方式来请求锁。示例代码如下所示。主线程先获取到了锁并一直持有，之后线程t1调用lockInterruptibly()请求锁，因为锁被主线程持有，所以，线程t1阻塞等待。主线程调用interrupt()函数向线程t1发起中断请求，线程t1响应中断请求，退出阻塞等待锁，并打印“I am interrupted”。
~~~java
public class Demo {
  private static Lock lock = new ReentrantLock();
  
  public static void main(String[] args) {
    Thread t1 = new Thread(new Runnable() {
      @Override
      public void run() {
        try {
          lock.lockInterruptibly();
        } catch (InterruptedException e) {
          System.out.println("I am interrupted");
          return;
        }
        try {
          System.out.println("I got lock");
        } finally {
          lock.unlock();
        }
      }
    });
    lock.lock();
    t1.start();
    t1.interrupt();
    lock.unlock();
  }
}
~~~
可中断锁方便关闭正在执行的线程。比如，Nginx服务器采用多线程来执行请求。当我们调用stop命令关闭Nginx服务器时，Nginx服务器可以采用中断的方式，将阻塞等待锁的线程中止，然后，合理的释放资源和妥善处理未执行完成的请求，以实现服务器的优雅关闭。

### 非阻塞锁
对于synchronized锁来说，一个线程去请求一个synchronized锁时，如果锁已经被另一个线程获取，那么，这个线程就需要阻塞等待。Lock锁提供了tryLock()函数，支持非阻塞的方式获取锁。如果锁已经被其他线程获取，那么，调用tryLock()函数会直接返回错误码而非阻塞等待。示例代码如下所示。非阻塞锁的实现原理非常简单。竞争锁失败的线程不放入队列排队即可实现非阻塞锁。
~~~java
public class Demo {
  private Lock lock = new ReentrantLock();
  
  public void useTryLock() {
    if (lock.tryLock()) {
      try {
        // ...执行业务代码...
      } finally {
        lock.unlock();
      }
    } else {
      // ...没有获取锁，执行其他业务代码...
    }
  }
}
~~~

### 可超时锁
除了提供不带参数的tryLock()函数之外，JUC Lock接口还提供给了带时间参数的tryLock()函数，支持非阻塞获取锁的同时设置超时时间。一个线程在请求锁时，如果这个锁被其他线程持有，那么这个线程会阻塞等待一段时间。如果超过了设定的超时时间，线程仍然没有获取到锁，那么tryLock()函数将会返回错误码而不再阻塞等待。示例代码如下所示。从示例代码中，我们还可以发现，tryLock()跟lockInterruptibly()一样，也可以被中断。这样是为了避免tryLock()阻塞过长时间。
~~~java
public class Demo {
  private Lock lock = new ReentrantLock();
  
  public void useTryLockWithTimeout() {
    boolean locked = false;
    try {
      locked = lock.tryLock(100, TimeUnit.MILLISECONDS);
    } catch (InterruptedException e) {
      System.out.println("I am interrupted");
    }
    if (locked) {
      try {
        // ...执行业务代码...
      } finally {
        lock.unlock();
      }
    } else {
      // ...没有获取锁，执行其他业务代码...
    }
  }
}
~~~

可超时锁的应用场景
在很多对响应时间比较敏感的系统中，比如直接面向用户的系统，从用户体验上说，请求失败给予提示，要远好于响应超时而没有反应。我们拿Tomcat等Web服务器来举例。Tomcat采用线程池的方式多线程执行用户请求。如果某个特殊请求不能并发执行，并且请求执行时间比较长，那么，请求的处理代码就需要加锁。当多个线程同时执行多个特殊请求时，有些线程就有可能因为迟迟无法获取到锁而无法执行请求。一方面，这样会导致用户请求超时，给用户带来不好的体验，另一方面，线程一直等待锁，长期被占用，无法执行其他任务，剩余可以执行用户请求的线程变少，从而导致系统的响应速度变慢。

为了解决以上问题，可以使用带超时时间的tryLock()函数来请求锁。如果在设定的超时时间内未获取到锁，那么，线程就中止执行用户请求，返回错误信息给用户。当然，这只是保护措施，毕竟，以上问题只有在无法并发执行的特殊请求集中大量到来时才会发生。

## 


