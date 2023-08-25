# JUC包
- Java.util.concurrent包
- Java的并发工具包
---

# 为什么需要JUC包
- synchronized性能不⾼
- wait/notify/notifyAll 太原始了，难⽤
- 不够灵活
- 只有悲观锁、排他锁，没有乐观锁、共享锁 

# JUC的改进
- 提⾼了性能
- 提供了多种场景下更⽅便的实现
- 易⽤


# Lock/Condition 
## Lock/Condition与synchronized/wait/notify机制的差别

## 锁的重⼊问题
- ### synchronized需要使用递归才能实现多次上锁 对比 ReentrantLock 可重入锁 
  - 对于一个ReentrantLock对象可以多次调用lock()方法上锁，但也需要多次调用unlock()方法解锁，否则其他线程无法持ReentrantLock对象
  - lock() 方法   当前线程持有锁数量加1，注意在上锁时必须要拿到这把锁否则就会进入休眠状态直到拿到这把锁  
  - trylock() 方法 如果当前线程已经持有此锁或其他线程不持有这把锁，则获取该锁，当前线程持有锁数量加1，并且该方法返回true。如果锁由另一个线程持有，则此方法将立即返回false。该方法会破坏公平性。
  - tryLock(long timeout, TimeUnit unit) 方法  如果在给定的参数timeout等待时间内没有被另一个线程持有并且当前线程未被中断或当前线程已经持有此锁，则获取该锁，当前线程持有锁数量加1,并且该方法返回true。如果锁由另一个线程持有，则此方法将使当前线程进入休眠状态，直到当前线程在超时时间之前持有这把锁或超过等待时间返回false。参数timeout小于零或等于零则不等待，参数unit 超时参数的时间单位。如果将此锁设置为使用公平的排序策略，有任何其他线程正在等待该锁，则不会获取可用锁。即不会破坏公平性
  - unlock() 方法 解锁 当前线程持有锁数量减一，当前线程持有锁数量为零时释放锁。注意在解锁时必须要拿到这把锁否则就会抛异常
  - 默认的ReentrantLock是非公平锁，即使用ReentrantLock()构造器创建的，如果需要使用公平锁则需要使用ReentrantLock(boolean fair)创建并且使传入的参数fair为true，fair为false则与默认的创建一样是非公平锁
## 更加灵活
  - 同⼀个锁可以有多个条件
  - 读写分离
  - 可以⽅便的实现更加灵活的优先级/公平性
    - 公平锁和⾮公平锁
      - 在多个线程同时竞争一个锁的时候使用公平的排序策略就是公平锁
      - 在多个线程同时竞争一个锁的时候使用随机选取即谁先拿到锁谁运行就是非公平锁
---


# CountDownLatch
  - 倒数闭锁：⽤于协调⼀组线程的⼯作。拦截线程，使线程进入等待当锁存器计数到零时唤醒被拦截的所有线程
  - 简单、明了、粗暴的API
    - CountDownLatch(int count) 构造器参数count确定了锁存计数的初始值
    - countDown() 当前锁存计数减1，如果当前锁存计数为零则什么也不发生
    - await() 使当前线程等待，直到锁存器计数到零则唤醒线程
    - await(long timeout, TimeUnit unit) 使当前线程等待，直到锁存器计数到零则唤醒线程或参数timeout所指定时间过去，参数unit是参数timeUnit的时间单位
    - getCount() 获取当前锁存计数器计数
---


# CyclicBarrier
  - 设置了屏障，必须等所有的线程执行到屏障了线程才能进行下一步操作
  - API更加简单粗暴
    - await() 等到所有线程都在此屏障上调用该方法了线程才能进行下一步操作
    - await(long timeout, TimeUnit unit) 等到所有线程都在此屏障上调用该方法了线程才能进行下一步操作或参数timeout所指定时间过去，参数unit是参数timeUnit的时间单位
---

# Semaphore
  - 信号量：规定一个资源在同一时间有多少线程可以使用
  - 信号量的获取和释放
    - avpuire() 从信号量获取资源的一个使用权限如果没有则当前线程进入休眠状态直到其他线程释放资源的使用权限
    - avpuire(int permits) 从信号量获取资源permits参数数量的使用权限如果没有则当前线程进入休眠状态直到其他线程释放资源的使用权限
    - release() 信号量释放当前资源的一个使用权限
    - release(int permits) 信号量释放当前资源permits参数数量的使用权限
--- 


# BlockingQueue/BlockingDeque
  - 传统的集合框架的操作要么正常返回，要么丢出异常
  - BlockingQueue/BlockingDeque提供⼀种「等待」的可能
  - 创建时确定内部大小，当队列满了在放东西进去就使当前线程进入休眠，直到出现可以放东西的空间为止
  - 单向队列和双端队列的实现
    - ArrayBlockingQueue
    - LinkedBlockingQueue
  - API阻塞操作：put/take
    - put()将指定的元素插入此队列，当队列满了在放东西进去就使当前线程进入休眠，直到出现可以放东西的空间为止
    - take()检索并除去此队列的头，如队列为空，请等待直到元素可用
---

# Future与ExecutorService
  - ⼀个多线程执⾏器框架，最常⽤的实现是线程池
  - 它屏蔽了线程的细节，提供了并发执⾏任务机制
  - 为什么需要线程池？
    - 线程的代价太昂贵
    - 我们应该避免使⽤Executors么？
  - Future代表⼀个「未来才会发⽣的事情」
    - Future本身是⽴即返回的
    - get()会阻塞并返回执⾏结果，并抛出可能的异常

# 线程池（ThreadPoolExecutor）详解
- 使⽤线程池完成多线程协同和任务分解
- Executors 工厂类提供的线程池创建方法有一些缺陷，因此在实际应用中不推荐使用，特别是在生产环境中，相反推荐使用手动 new ThreadPoolExecutor 手动控制参数
- 线程池的参数们
  - corePoolSize ：核⼼线程数量
  - maximumPoolSize ：最⼤线程数量
  - keepAliveTime ：当线程数大于核心，这是多余空闲线程的最大时间数将在终止前等待新任务。
  - unit ：keepAliveTime参数的时间单位
  - workQueue ：工作队列，线程需要处理的任务
  - threadFactory ：线程的⼯⼚
  - handler ：任务实在太多的拒绝策略

 

 

