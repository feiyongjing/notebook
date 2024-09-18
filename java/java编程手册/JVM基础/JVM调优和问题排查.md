## JVM性能优化

### 多久一次、一次多久FullGC和YoungGC才算正常？
因为大部分系统的性能压力都很小，内存充足，GC频率很低，默认的JVM设置就已经够用，因此，也就没有进行JVM性能优化的必要。这就导致大部分工程师对JVM性能优化都很陌生，甚至连多久进行一次FullGC或YougGc以及一次FullGC或YoungGC耗费多长时间才算合理都搞不清楚。

### JVM性能指标
既然要对JVM进行性能优化，那么首先要知道如何评估JVM的性能。实际上，应用程序直接关注的性能指标主要就两个：GC频率和GC时间。当然，除了这两个性能指标之外，JVM内部还有很多更加细致的性能指标，如下所示，这些内部性能指标综合起来决定了GC频率和GC时间。
- 年轻代中对象的增长速率
- 每次YoungGC之后存活对象大小
- 每次YoungGC之后进入老年代的对象大小
- 老年代对象的增长速率

## JVM参数设置
以上罗列的性能指标受JVM参数的影响。合理的设置JVM参数可以将GC效率发挥到极致，尽可能地减少GC对应用程序的影响。而不合理的设置JVM参数有可能引起JVM性能问题，比如频繁的FullGC、GC时间过长等。
JVM参数一般有3种类型：标准参数（以-开头，比如-version）、X参数（以-X开头，比如-Xint、-Xms2048m）、XX参数（以-XX开头，比如-XX:+PrintGCDetails、-XX:PermSize=512m），这类参数的稳定性依次下降，也就是说，在Java版本更新的过程中，标准参数很少改动，X参数有可能会改动，XX参数改动的可能性比较大。
JVM参数非常多，有上百个，但是，对绝大部分参数来说，默认的设置便是最普适、最合理的设置。除非真的有必要，否则，不应该主动去设置这些参数。常用的JVM GC参数只有几个，主要集中在内存分配和垃圾回收器的设置这两个方面，具体如下所示。

### 设置堆的大小
- -Xms：堆内存的初始大小
- -Xmx：堆内存的最大大小
一般情况下，会把-Xms和-Xmx设置为相同的值，以避免堆大小的调整而引起的性能损耗

### 设置年轻代和老年代的大小
- -Xmn：年轻代大小
- -XX:NewSize：年轻代的初始大小
- -XX:MaxNewSize：年轻代的最大大小
- -XX:NewRatio：年轻代与老年代的大小比值，比如-XX:NewRatio=4表示年轻代与老年代的大小比值是1:4，年轻代占整个堆大小的1/5
设置年轻代大小的方式有多种：使用-Xmn、使用-XX:NewSize和-XX:MaxNewSize以及使用-XX:NewRatio。不需要显式地去设置老年代大小，将堆大小减去年轻代大小便可得到老年代大小

### 设置永久代或元空间的大小
- -XX:PermSize：永久代的初始大小
- -XX:MaxPermSize：永久代的最大大小
- -XX:MetaspaceSize：元空间的初始大小
- -XX:MaxMetaspaceSize：元空间的最大大小
前两个参数仅在Java1.7及其以前版本中有效，后两个参数仅在Java1.8及其以后版本中有效

### 设置Eden区和Survivor区的大小
-XX:SurvivorRatio：Survivor区跟Eden区的大小比例，比如-XX:SurvivorRatio=8表示一个Survivor区跟Eden区的大小比例是1:8，也就是说，Eden区占年轻代大小的8/10，两个Survivor区分别占年轻代大小的1/10

### 设置线程栈的大小
-Xss：每个线程的栈大小，HotSpot JVM不区分虚拟机栈和本地方法栈，使用一个栈同时存储Java方法和本地方法的栈帧，因此这里只有一个栈大小的设置参数。线程栈大小默认为512KB或1MB。除非系统在运行的过程中出现非代码因素导致的StackOverflow，才需要调整线程栈的大小，否则，一般使用默认的线程栈大小设置即可

### 设置垃圾回收器
- -XX:+UseSerialGC：使用Serial垃圾回收器
- -XX:+UseParallelGC：使用Parallel垃圾回收器
- -XX:+UseConcMarkSweepGC：使用CMS垃圾回收器
- -XX:+UseG1GC：使用G1垃圾回收器

## JVM性能预估调优
刚刚介绍了常用的JVM参数，那么，如何设置合理的JVM参数呢？
实际上，大部分情况下，只需要对常用的JVM参数，预设一些经验值，然后根据线上或者压测的情况，再做优化调整即可。当然，也可以事先预估一下系统对内存的使用情况，比如每秒钟产生多少对象，对象的生存周期等等，然后，依据期望的GC频率和GC时间，有针对性地设置JVM参数。具体如何来做通过一个例子来讲解。
对系统中调用频率最高的几个接口进行分析，罗列出执行每个接口请求所需要创建的对象大小，然后取平均值便粗略得到了执行一个接口请求所创建的对象的平均大小，假设这个值为2KB。预估系统的QPS是500，那么，就可以得知，系统每秒钟产生的对象大小大约1MB（500*2KB）。如果年轻代中Eden区和From Survivor区的总大小为1GB，那么，填满它们大约需要1000秒（即约16分钟），也就是说，系统每隔16分钟就会进行一次YoungGC。
再假设执行YoungGC会回收90%的年轻代空间，剩余100MB（1GB*10%）存活对象会被复制到To Survivor区。前面讲到，根据动态年龄判定机制，To Survivior区中的对象所占空间超过50%，就会导致部分对象进入老年代。为了尽量避免对象进入老年代而引发FullGC，需要保证To Survivor区大小至少为200MB。因此，年轻代大小应该设置为1.2GB。
再假设这100MB存活对象中，有10%的对象的生命周期特别长，会经过多次YoungGC之后进入老年代。也就是说，每次YoungGC会有大约10MB的对象进入老年代。假设老年代的大小约为1.8GB，并且每次垃圾回收只能回收老年代50%的空间，那么，每经过大约90次（1.8GB * 50% / 10MB）YoungGC就会执行一次FullGC，由此推出FullGC的时间间隔大约为24小时（90 * 16分钟 / 60）。
根据以上假设和分析，设置JVM参数，如下所示。这里假设系统名称为WebServer。因为系统对接口的响应时间比较敏感，因此，使用CMS垃圾回收器。年轻代的大小为1.2GB，为了保证To Survivor区大小至少为200MB，将-XX:SurvivorRatio设置为4。除此之外，存放类等元信息的元空间大小设置为512MB。线程栈大小使用默认值。
~~~shell
java -Xms3g -Xmx3g -Xmn1.2g -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m -XX:SurvivorRatio=4 -XX:+UseConcMarkSweepGC -jar WebServer.jar
~~~

不过，上述分析过程包含大量的假设，这就导致最终得到的JVM参数值可能并不准确，甚至完全没有意义，反倒不如经验值合理。实际上，初始JVM参数如何设置关系并不大，毕竟在项目上线或者压测的一段时间内，还需要密切关注JVM的表现，并且基于jstat等工具得到的JVM性能统计数据，进一步做性能调优。调优之后的结果才是最重要的。那么，具体如何进行JVM性能调优呢？
JVM性能调优的方向是减少GC频率和GC时间，特别是FullGC频率和FullGC时间。YoungGC时间往往都比较短，正常情况下，一般都在几毫秒或几十毫秒，对应用程序的影响很小，因此，YoungGC稍微频繁一点问题也不大。相比而言，FullGC要慢很多，正常情况下，一般都在几十毫秒或几百毫秒，对应用程序的影响较大。需要通过调整JVM参数，比如增大年轻代大小、增大Survivor区大小，让对象尽量在年轻代就被回收掉，减少老年代中对象的增长速率，从而降低FullGC频率。除此之外，增大老年代大小，也可以降低FullGC频率，当然这又会增大FullGC时间。
一般来讲，如果堆不是很大，那么，在没有长期存活的大对象和内存泄漏的情况下，应用CMS垃圾回收器并调节年轻代、老年代、Survivor区等的内存分配，完全可以将FullGC时间优化到合适的范围。如果实在不行的话，可以选择GC时间可控的G1垃圾回收器。
实际上，多久执行一次GC（YoungGC或FullGC）、一次GC（YoungGC或FullGC）耗时多久才算合理，并没有唯一的标准答案。具体值还是要看系统需求来定。如果系统并不在意STW时间，比如离线系统，那么，FullGC频繁一点、时间久一点，也问题不大。如果系统是响应时间比较敏感的系统，比如接口服务，那么，FullGC时间超过1秒就是不可接受的。
实际上，大部分情况下，经验值设置就已经满足绝大部分系统的需求，并不需要刻意的对JVM参数进行调优。只有当通过监控发现，GC的频率过大或GC时间过长，严重影响系统性能时，才有必要对JVM参数进行调优

## JVM问题排查
### JVM性能监控和分析工具
用于JVM性能监控和分析的工具有很多，常见的有jstat、JConsole、VisualVM、jmap、MAT以及一些JVM参数。按照功能，将它们分为3类：GC统计信息监控、GC详细日志分析、JVM内存快照获取和分析，接下来，就按照类别简单介绍一下这些工具。

## GC统计信息监控
### jstat是命令行工具，JConsole、VisualVM是可视化工具。JConsole、VisualVM不仅可以监控GC的情况，还可以监控系统资源（如CPU、内存等）的使用情况、线程的执行情况等。这里重点介绍一下更加简单常用的jstat。
jstat命令使用起来非常简单，如下
~~~shell
# 提取通过jps命令查找到要监控的JVM进程ID，然后执行如下命令
jstat -gcutil [pid] [time-interval]
# time-interval为每次输出GC统计信息的时间间隔，以毫秒为单位

# 在以上jstat输出的统计信息中，各列的含义如下所示。
#   S0：表示Survivor0的内存使用率；
#   S1：表示Survivor1的内存使用率；
#   E：表示Eden区的内存使用率；
#   O：表示老年代的内存使用率；
#   M：表示Metaspace的内存使用率；
#   YGC：YoungGC的次数；
#   YGCT：YoungGC的总耗时；
#   FGC：FullGC的次数；
#   FGCT：FullGC的总耗时；
#   GCT：GC的总耗时。
# 通过以上统计信息，可以获得GC的各项重要指标，比如YoungGC频率、FullGC频率、YoungGC耗时、FullGC耗时、年轻代的垃圾回收率、老年代的垃圾回收率、年轻代的对象增长速率、老年代的对象增长速率等，以此来检查JVM是否存在问题。
~~~

### GC详细日志分析：JVM参数-XX:+PrintGCDetails等
从上述讲解，可以发现，jstat只能输出GC的统计信息，如果希望获得GC的详细情况，那么，就需要使用如下JVM参数
~~~
-XX:+PrintGCDetails：打印详细GC日志
-Xloggc:./logs/gc.log：详细GC日志存储的位置
~~~

在设置以上JVM参数之后，注意，不同版本的JVM和不同的垃圾回收器输出的GC日志格式会有所不同，这里显示的是Java8和CMS垃圾回收器对应的GC日志格式。
以上输出的日志可以粗略的分为两类：ParNew日志和CMS日志。先来看ParNew日志。拿上图中的第一条日志来举例讲解。
~~~
[GC (Allocation Failure) 4.527: [ParNew: 48243K->6054K(55296K), 0.0101725 secs] 48243K->11277K(96256K), 0.0103798 secs] [Times: user=0.08 sys=0.01, real=0.01 secs]
~~~
对上述ParNew日志所包含的信息逐一解读，如下所示：
- GC（Allocation Failure）：表示GC是因为内存分配失败触发的
- 4.527：表示GC发生的时间（距离JVM启动所经历的秒数）
- [ParNew：48243K->6054K(55296K)，0.0101725 secs]：表示GC的情况。ParNew表示GC类型为Parallel New，48243K和6054K表示GC前后年轻代中对象的大小。55296K表示年轻代的总大小。0.0101725表示GC耗时
- 48243K->11277K(96256K)，0.0103798 secs：48243K和11277K表示本次GC前后整个堆中对象的大小变化。96256K表示整个堆的总大小
- [Times: user=0.08 sys=0.01, real=0.01 secs]：表示GC具体的耗时分析。user=0.08表示GC线程使用CPU的时间，sys=0.01表示系统调用的时间，real=0.01表示墙上时钟，也就是从GC开始到结束所经历的时间。注意，在多核多线程情况下，user时间为每个线程所使用的CPU时间之和，因此，它的大小很有可能超过real时间的大小。

接下来，再来看下CMS日志。拿其中一条CMS日志举例讲解，如下所示：
~~~
25.933: [GC (CMS Initial Mark) [1 CMS-initial-mark: 22706K(40960K)] 29330K(96256K), 0.0002184 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

25.934: [CMS-concurrent-mark-start]
25.935: [CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

25.935: [CMS-concurrent-preclean-start]
25.935: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

25.935: [GC (CMS Final Remark) [YG occupancy: 6624 K (55296 K)]25.935: [Rescan (parallel) , 0.0004028 secs]25.935: [weak refs processing, 0.0000226 secs]25.935: [class unloading, 0.0002652 secs]25.936: [scrub symbol table, 0.0003544 secs]25.936: [scrub string table, 0.0001626 secs][1 CMS-remark: 22706K(40960K)] 29330K(96256K), 0.0013012 secs] [Times: user=0.01 sys=0.00, real=0.00 secs]

25.936: [CMS-concurrent-sweep-start]
25.936: [CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]

25.936: [CMS-concurrent-reset-start]
25.937: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
~~~
CMS日志包含的信息比ParNew日志要多很多，其中，CMS Initial Mark、CMS-concurrent-mark、CMS Final Remark、CMS-concurrent-sweep分别对应并发垃圾回收的四个阶段：初始标记、并发标记、重新标记、并发清理。除此之外，CMS日志中的CMS-concurrent-preclean和CMS-concurrent-reset为更加细化的并发垃圾回收阶段，负责CMS内部数据结构的调整工作，这里就不展开详细讲解了。

### JVM内存快照获取和分析：JVM参数-XX:+HeapDumpBeforeFullGC、-XX:+HeapDumpOnOutOfMemoryError、jmap、MAT、jhat等
以上讲到都是监控GC情况的工具，但是，当JVM出现问题时，比如OOM、频繁GC，如果希望得知当前堆中存储的对象情况，比如哪些对象占据了大量的堆内存，就需要将当下的内存快照dump出来，然后利用工具来查看和分析。
常用的dump堆内存快照的方法有两种，一种是使用JVM参数，另一种是使用jmap命令行工具。具体如下所示。dump出来的堆内存快照为二进制文件，需要通过工具来查看，常用的查看工具有MAT、jhat等。
方法一：使用JVM参数
~~~
-XX:+HeapDumpBeforeFullGC
-XX:HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=目录
~~~

方法二：使用jmap命令行工具
~~~
jmap -dump:format=b,file=文件名 [pid] 
~~~


## JVM常见问题一：OOM
当应用程序申请不到足够的内存空间，并且，即便JVM通过垃圾回收也无法释放出足够的内存空间时，JVM便会抛出java.lang.OutOfMemoryError异常，这种情况就称为OOM（Out Of Memory），即内存溢出。内存溢出又分为永久代（元空间）的内存溢出和堆的内存溢出，分别对应以下两种错误信息。
~~~
java.lang.OutOfMemoryError: PermGen space(或 MetaSpace)
java.lang.OutOfMemoryError: Java heap space
~~~

导致内存溢出的常见原因有如下几种。
- 设置的堆或永久代（元空间）的大小太小
- 一次性创建过多的对象，比如通过SQL查询全表数据
- 应用程序使用完成的对象没有被及时释放，导致对应的内存无法被回收，长期积累便会导致内存耗尽。把这种情况叫做内存泄露

关于内存泄露，举例讲解。在下述代码中，Stack在使用的过程中会出现内存泄露问题，请你思考下这是为什么呢？
~~~java
public class Stack {
  private Object[] array;
  private int capacity;
  private int top;

  public Stack(int capacity) {
    this.capacity = capacity;
    this.top = 0;
    this.array = new Object[capacity];
  }

  public void push(Object obj) {
    array[top++] = obj;
  }

  public Object pop() {
    return array[--top];
  }
}
~~~

当执行上述pop()函数时，pop()函数只是将top值减一并返回对应的对象，并没有删除array数组跟对象之间的引用关系。因此，对象即便已经使用完成，也无法被回收。为了解决这个内存泄露问题，只需要对pop()函数稍作修改，将引用关系删除即可，具体如下所示。
~~~java
public Object pop() {
  Object obj = array[top-1];
  array[top-1] = null; //删除引用关系
  top--;
  return obj;
}
~~~

了解了OOM问题产生的原因，再来看下如何排查OOM问题
当JVM出现OOM问题时，应用程序的对应表现一般是无法继续执行，如果应用程序是接口系统，那么接口将出现大量503错误。这时通过查看日志，便会发现大量的java.lang.OutOfMemoryError错误信息。为了排查出到底哪些对象长期存在并大量占用内存，需要通过jmap或JVM参数获取堆内存快照，并通过MAT等工具来查看和分析。
举例解释上述排查过程。示例代码如下所示。将堆大小设置为100MB。在OOMDemo类中，代码循环执行api()函数，api()函数创建大小约1MB大小的Student对象并放入list中长期持有，这样就会导致堆内存很快被耗尽，随之发生OOM。
~~~shell
java -Xms100m -Xmx100m -Xmn60m -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./ OOMDemo
~~~

~~~java
public class OOMDemo {
  private static final List<Student> list = new ArrayList<>();
  public static void main(String[] args) {
    for(;;) {
      api();
    }
  }

  public static void api() {
    Student stu = new Student();
    list.add(stu);
  }

  private static class Student {
    private byte[] data = new byte[1024*1024];
  }
}
~~~

当发生OOM时，JVM会自动dump堆内存并生成快照文件（比如java_pid31969.hprof），将快照文件导入MAT工具，查看内存占用的详细情况，可以发现，怀疑内存泄露的数据主要集中在OOMDemo的list成员变量中。以此再去分析OOMDemo的代码，看是否真正存在内存泄露，还是创建了太多长期存在的对象。如果两者都不是，那么，这就说明并非代码问题，那么，尝试调大堆内存的大小来看是否能解决此问题。

## JVM常见问题二：频繁GC
一般来讲，在OOM出现之前，JVM会先出现频繁GC的现象。频繁GC包括频繁YoungGC和频繁FullGC。单纯的频繁YoungGC往往是由年轻代空间太小导致的，只需要适当增大年轻代的大小即可解决这个问题。
你可能会说，增大年轻代大小会不会导致YoungGC时间增长呢？实际上并不会。首先，年轻代采用的是标记-复制垃圾回收算法。标记（即可达性分析）只对存活对象进行遍历，而复制也只会对存活对象进行复制。因此，YoungGC时间只与存活对象的数量有关，与年轻代大小无关。其次，大部分对象都是朝生夕死，年轻代空间增大，YoungGC时间间隔增长，这会导致更多的对象在GC前就死亡，年轻代中的存活对象并不会明显增多。综合以上两点，YoungGC时间并不会随着年轻代的增大而增长。

相对于频繁YoungGC，频繁FullGC会引发更加严重的问题，且解决起来更加复杂。因为FullGC更加消耗CPU资源并且STW时间更长，所以，在发生频繁FullGC时，CPU利用率一般会飙升，并且会出现应用程序变慢的情况（比如接口请求处理速度变慢甚至大量超时）。
触发FullGC的主要原因是老年代空间不足。前面已经总结过，老年代的对象一般来源于长期存活的对象、大对象、空间分配担保。接下来，从这3个对象来源来分析频繁GC发生的原因。
1. 长期存活的对象
如果应用程序创建的长期存活的对象比较多，那么，可以适当调大老年代的大小，以减少FullGC的频率。不过，这种情况并不常见，大部分应用程序并不会创建太多的长期存活的对象。实际上，内存泄露才是导致对象长期存活无法回收的主要原因。如果每次FullGC回收率很低，释放出来的空间很少，那么，这就说明是存在内存泄露了。频繁FullGC一段时间之后，JVM便会出现OOM。
2. 大对象
前面讲到，大对象会直接进入老年代。过多的大对象是引起频繁FullGC的最常见的原因之一。比如，在某个接口中执行了未分页SQL，一次性加载过多数据到内存中。在高并发下，接口被大量调用，就会导致大量大对象被创建，从而导致老年代空间不足，引发频繁FullGC。定位此种频繁FullGC发生的原因，需要在FullGC前（设置JVM参数-XX:+HeapDumpBeforeFullGC）dump堆内存快照，分析占用堆内存比较多的是哪个对象，以此来定位问题代码。
3. 空间分配担保
前面讲到，在执行YoungGC时，如果To Survivor空间不足，JVM会触发空间分配担保，将对象存储到老年代。因此，如果每次YoungGC，To Survivor都被占满，那么，就要考虑增大To Survivor区，避免空间分配担保，减少进入老年代的对象数量。

## JVM常见问题三：GC时间过长
导致GC时间过长的常见原因有：堆内存过大、Concurrent Mode Failure、操作系统swap
1. 堆内存过大
前面讲到，年轻代使用标记-复制垃圾回收算法，并且，年轻代空间增大并不会导致存活对象增多，因此，YoungGC时间跟年轻代的大小无关，但是，老年代使用标记-整理或标记-清除垃圾回收算法，并且，老年代空间增大会导致存活对象增多，因此，FullGC时间跟老年代的大小有关。老年代过大会导致FullGC时间过长。针对比较大的堆内存，应该选择GC时间可控的G1垃圾回收器，或者在一台大物理内存的机器上部署多个JVM以减小单个堆内存的大小。
2. Concurrent Mode Failure
前面讲到，CMS垃圾回收器采用并发垃圾回收算法，在垃圾回收的某些阶段，应用程序可以与之并发执行。应用程序的执行需要堆内存，因此，JVM在执行垃圾回收前，会预留一定的堆内存空间。但是，在执行垃圾回收的的过程中，如果预留空间不足，应用程序无法继续执行，那么，JVM便会抛出Concurrent Mode Failure错误，并且，暂停CMS垃圾回收器的执行，改为STW时间更长的Serial Old垃圾回收器。垃圾回收器的中止和切换势必会增长FullGC时间。如果在GC详细日志中（通过设置JVM参数-XX:+PrintGCDetails得到）发现大量Concurrent Mode Failre字样，那么，就需要通过减小JVM参数-XX:CMSInitialOccupancyFraction的值来调大预留空间的大小。
3. 操作系统swap
swap是操作系统中的概念。当物理内存不足时，操作系统会将物理内存中的部分不活跃的数据放入磁盘，当这部分数据重新被使用时，再从磁盘加载到物理内存中。这种数据在物理内存和磁盘之间换入换出的机制，就叫作swap。swap涉及磁盘I/O操作，非常影响进程的性能。如果设置的JVM堆内存大小超过物理内存大小，或者多个应用程序争用有限的物理内存，那么，这就有可能触发swap而导致GC时间增长。解决这个问题的方法也很简单，尽量保证JVM堆大小不要超过物理内存的大小，并且为操作系统和其他软件预留充足的物理内存，比如物理内存有8GB，设置JVM堆大小为6GB，预留2GB给操作系统和其他并发运行的软件。




