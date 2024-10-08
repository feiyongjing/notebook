# Lock类
~~~java
//当前线程尝试获取锁，如果锁被其他线程占有，则等待直到获取到锁
void lock();​  // 阻塞获取锁

 
// 当前线程尝试获取锁，成功则立即返回true，失败则立即返回false，不会进行等待
boolean tryLock();​   // 非阻塞获取锁


// 当前线程在time时间（unit 指时间单位）内尝试获取锁，成功则立即返回true，超过时间则立即返回false
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

 
// 当前线程尝试获取锁，但是获取锁失败后进入等待状态后，其他线程可以调用当前线程的Thread对象的  interrupt​() 方法来中断等待状态
void lockInterruptibly() throws InterruptedException;


// 当前线程释放锁，如果当前线程不持有锁直接调用这个方法会抛出UncheckedExection
void unlock();

 
// 创建一个与当前重入锁绑定的Condtion实例返回
Condition newCondition();

~~~
 
# Condition类
~~~java
//使当前线程加入 await() 等待队列中，并释放当锁，当其他线程调用signal()会重新请求锁。与Object.wait()类似。
void await() throws InterruptedException;

//调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出IllegalMonitorStateException。
//调用该方法后，结束等待的唯一方法是其它线程调用该条件对象的signal()或signalALL()方法。等待过程中如果当前线程被中断，该方法仍然会继续等待，同时保留该线程的中断状态。
void awaitUninterruptibly();

// 调用该方法的前提是，当前线程已经成功获得与该条件对象绑定的重入锁，否则调用该方法时会抛出IllegalMonitorStateException。
//nanosTimeout指定该方法等待信号的的最大时间（单位为纳秒）。若指定时间内收到signal()或signalALL()则返回nanosTimeout减去已经等待的时间；
//若指定时间内有其它线程中断该线程，则抛出InterruptedException并清除当前线程的打断状态；若指定时间内未收到通知，则返回0或负数。
long awaitNanos(long nanosTimeout) throws InterruptedException;

//与await()基本一致，唯一不同点在于，指定时间之内没有收到signal()或signalALL()信号或者线程中断时该方法会返回false;其它情况返回true。
boolean await(long time, TimeUnit unit) throws InterruptedException;

//适用条件与行为与awaitNanos(long nanosTimeout)完全一样，唯一不同点在于它不是等待指定时间，而是等待由参数指定的某一时刻。
boolean awaitUntil(Date deadline) throws InterruptedException;

//唤醒一个在 await()等待队列中的线程。与Object.notify()相似
void signal();

//唤醒 await()等待队列中所有的线程。与object.notifyAll()相似
void signalAll();

 
~~~
