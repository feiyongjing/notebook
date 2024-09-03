# Java I/O
I/O全称为Input/Output，中文为输入/输出。在计算机中，常用的I/O设备有硬盘、网络、键盘、显示器等。在操作系统层面，I/O系统有文件、网络、标准输入和输出（对应键盘和显示器）、管道等。Java提供的I/O类库就是用来读写这些I/O系统的。

在JDK1.4之前，Java引入了java.io类库，用来支持基本的I/O操作。在JDK1.4及其之后，Java引入了java.nio类库，用来支持非阻塞I/O模型的开发。
在JDK7中，Java对java.nio类库进行了升级，引入了更多的类，用来支持异步I/O模型的开发。

java.io类库分类，如下
- 按照数据流向来分类：输入流和输出流
- 按照数据流的读写单位来分类：字节流和字符流
- 按照设计模式的角度来分类：原始类和装饰器类

## java.io类库

### 输入流和输出流
输入流：InputStream、Reader
输出流：OutputStream、Writer

所谓输入流，指的是将文件、网络、标准输入（System.in）、管道中的数据，输入到内存中。所谓输出流，指的是将内存中的数据输出到文件、网络、标准输出（System.out、System.err）、管道中。

输入流的读取方式如下示例代码所示。因为我们通过try-catch-resources语句打开InputStream，所以，在try代码块结束后，JVM会自动调用InputStream的close()函数关闭输入流。为了简化代码编写，后续示例代码均省略异常的捕获处理。
~~~java
try (InputStream in = new FileInputStream("/Users/in.txt")) {
  byte[] data = new byte[1024];
  while (in.read(data) != -1) {
    //处理data数组...
  }
} catch (FileNotFoundException e) {
  e.printStackTrace();
} catch (IOException e) {
  e.printStackTrace();
}
~~~

输出流的写入方式，如下示例代码所示。
~~~java
OutputStream out = new  FileOutputStream("/Users/out.txt")；
byte[] data = new byte[128];
out.write(data);
~~~

### 字节流和字符流
字节流：InputStream、OutputStream
字符流：Reader、Writer

所谓字节流，指的是按照字节为单位从输入流中读取数据，或者将数据写入输出流。所谓字符流，指的是按照字符为单位从输入流中读取数据，或者将数据写入输出流。实际上，相比字节流，字符流只是多了一个字符编码转换的环节。

从java.io类图中可以发现，Java分别为字符流和字节流设计了两套类。这两套类的代码实现有些重复。毕竟I/O读写操作都是相同的，唯一的区别只是数据解析的方式不同。因此，为字节流和字符流设计两套类完全是没有必要的。java.nio利用“组合优于继承”的设计思想，引入Channel和Buffer的概念，移除了大量重复代码。关于这一点，在java.nio中有详细讲解。

### 原始类和装饰器类
java.io类库的设计应用了装饰器设计模式。从设计模式的角度，我们可以将java.io类库中的类分为原始类和装饰器类。
装饰器类和原始类的区别在于，装饰器类是对原始类的功能增强，不能独立使用。比如，FileInputStream为原始类，可以独立使用，BufferedInputStream为装饰器类，支持缓存功能，不能独立使用，必须嵌套原始类或其他装饰器类。示例代码如下所示。
~~~java
InputStream in = new FileInputStream("/Users/in.txt"));
InputStream bin = new BufferedInputStream(in);
byte[] data = new byte[1024];
while (bin.read(data) != -1) {
    //处理data数组...
}
~~~

## IO类介绍
1. 文件IO类
跟文件读写相关的类有FileInputStream、FileOutputStream、FileReader、FileWriter。前面已经给出了一些文件读写的示例代码。这里就不再赘述。

2. 网络IO类
实际上，java.io类库并没有提供专门的类用于网络I/O的读写，而是直接复用了InputStream类和OutputStream类来读写网络I/O。除此之外，单独使用java.io类库也并不能完成网络编程，需要借助java.net类库的配合。java.net类库用来管理网络连接，比如创建连接、关闭连接、监听连接等。java.io类库只负责读写已经建立的网络连接。示例代码如下所示。java.io类库无法实现复杂的网络模型。java.nio类库便是因此而产生的。
~~~java
// Socket类位于java.net包中
Socket socket = new Socket("127.29.2.4", 8090);
OutputStream out = socket.getOutputStream();
out.write("hi~".getBytes());

InputStream in = socket.getInputStream();
byte[] data  = new byte[1024];
while (in.read(data) != - 1) {
  // do something
}
~~~

在java.io的类库中，InputStream、OutputStream是所有字节流类的父类，它既可以读写文件，也可以读写网络等其他I/O。这充分体现了“抽象”的设计思想。尽管深入到硬件层面，各个I/O设备的读写方式各不相同，但是，上层应用开发并不关系底层实现细节，大部分I/O设备的访问都可以抽象为打开、读、写、关闭等这几个操作。因此，Java将所有的I/O设备都抽象为“Stream（流）”，并设计了一套统一的接口。从而对于不同I/O设备的读写，我们可以使用同样的代码实现，代码更加统一、简洁。

3. 内存IO类
跟内存读写相关的类有：ByteArrayInputStream、ByteArrayOutputStream、CharArrayReader、CharArrayWriter、StringReader、StringWriter。我们将内存看做一种特殊的I/O系统，也可以像文件一样，当作Stream来读写。这些类的主要作用是实现兼容。比如，我们使用第三方类库中的某个函数，来处理byte数组中的数据，但这个函数的输入参数是InputStream类型的，为了兼容这个函数的定义，我们就可以将待处理的byte数组，封装成ByteArrayInputStream对象，再传递给这个函数处理，如下代码所示。
~~~java
byte[] source = "阿巴斯崇拜".getBytes();
InputStream in = new ByteArrayInputStream(source);
// 接下来就可以跟处理其他InputStream一样处理source了
~~~

在编写单元测试时，这些内存读写类也非常有用。如下代码所示，如果我们要为readFromFile()函数编写单元测试代码，那么，我们不需要创建包含测试数据的文件。使用ByteArrayInputStream，我们可以在内存中构建测试数据，这样就方便了很多。
~~~java
// 待测试函数
public int readFromFile(InputStream inputStream) { ... }

// 测试代码
public void test_readFromFile() {
  byte[] testData = new byte[512];
  //...构建测试数据，填入testData数组...
  InputStream in = new ByteIntputStream(testData);
  int res = readFromFile(in);
  //...assert...判断返回值是否符合预期...
}
~~~

4. 管道IO类
跟管道读写相关的类有PipedInputStream、PipedOutputStream、PipedReader、PipedWriter。这里的管道跟Unix操作系统中的管道不同。Unix操作系统中的管道是进程间的通信工具，而这里的管道是线程间的通信工具。一个线程通过PipedOutputStream写入数据，另一个线程可以通过PipedInputStream读取数据，示例代码如下所示。尽管Java已经提供了很多线程间通信的方式，比如共享变量，但是，一般来说，对于两个线程之间非对象的原始数据的传输，我们更倾向于使用管道来实现。示例代码如下所示。
~~~java
PipedOutputStream out = new PipedOutputStream();
PipedInputStream in = new PipedInputStream(out);
new Thread(new Runnable() {
  @Override
  public void run() {
    try {
      out.write("Hi 张三~".getBytes());
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}).start();

new Thread(new Runnable() {
  @Override
  public void run() {
    byte[] buffer = new byte[512];
    try {
      in.read(buffer);
      System.out.println(new String(buffer));
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}).start();
~~~

5. 标准输入输出
在操作系统中，一般会有三个标准I/O系统：标准输入、标准输出、标准错误输出。标准输入对应I/O设备中的键盘，标准输出和标准错误输出对应I/O设备中的屏幕。在Java中，我们使用System.in来处理标准输入，它是一个定义在System类中的静态InputStream对象。我们使用System.out和System.err来处理标准输出和标准错误输出，它们都是定义在System类中的PrintStream对象。PrintStream为装饰器类，需要嵌套OutputStream来使用，支持按照某种格式输出数据，待会会讲到。System.in、System.out、System.err的示例代码如下所示。
~~~java
Scanner s = new Scanner(System.in);
System.out.println("echo: " + s.nextLine());
//System.err显示的字符串为红色，以表示出错
System.err.println("echo: " + s.nextLine());
~~~

## 装饰器类分类介绍
装饰器类用于增强原始类的功能。我们按照功能的不同分类讲解java.io类库中的装饰器类。

### 支持读写缓存功能的装饰器类
支持读写缓存功能的装饰器类有BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter。这4个类的作用非常相似。我们拿BufferedInputStream、BufferedOutputStream举例讲解。

对比InputStream，BufferedInputStream会在内存中维护一个8192字节大小的缓存。当读取数据时，如果缓存中没有足够的数据，那么，read()函数会向操作系统内核请求数据，读取8192字节大小的数据存储到缓存中，然后read()函数再从缓存中返回需要的数据量。如果缓存中有足够多的数据，那么，read()函数直接从缓存中读取数据，不再请求操作系统。

向操作系统内核请求数据，需要使用系统调用，引起用户态和内核态的切换，是非常耗时的。读取相同大小的数据，相比于普通的InputStream，BufferedInputStream请求操作系统内核的次数更少。不过，如果read()函数每次请求的数据量都大于等于8192字节，那么，缓存就不起作用了。

BufferedInputStream的示例代码如下所示。如果文件大小是8192字节，那么，读取文件中的全部数据需要调用8次read()函数，但因为缓存的存在，所以，整个读取过程仅需要向操作系统内核请求一次数据。
~~~java
InputStream in = new FileInputStream("/Users/in.txt"));
InputStream bin = new BufferedInputStream(in);
byte[] data = new byte[1024];
while (bin.read(data) != -1) {
    //处理data数组...
}
~~~

同理，针对OutputStream，java.io类库提供了BufferedOutputStream，用来缓存写入的数据。当写入的数据积攒到一定量（默认为8192字节）时，BufferedOutputStream再一次性将其写入操作系统内核缓冲区，从而减少系统调用次数，提高程序的性能。

### 支持基本类型数据读写的装饰器类

DataInputStream支持将从输入流中读取的数据解析为基本类型（byte、char、short、int、float、double等）。DataOutputStream类支持将基本类型数据转化为字节数组写入输出流。示例代码如下所示。
~~~java
DataOutputStream out = new DataOutputStream(new FileOutputStream("/Users/a.txt"));
out.writeInt(12);
out.writeChar('a');
out.writeFloat(12.12f);
out.close();

DataInputStream in = new DataInputStream(new FileInputStream("/Users/a.txt"));
System.out.println(in.readInt());
System.out.println(in.readChar());
System.out.println(in.readFloat());
in.close();
~~~
调用DataOutputStream的readChar()、writeChar()函数，我们也可以按字符为单位读取、写入数据，但跟字符流类不同的地方是，DataOutputStream类一次只能处理一个字符，而字符流类可以处理char数组，并且字符流类提供的函数更多，功能更加丰富。

### 支持对象读写的装饰器类

ObjectInputStream支持将从输入流中读取的数据反序列化为对象，ObjectOutputStream支持将对象序列化之后写入到输出流。示例代码如下所示。
~~~java
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("/Users/a.txt"));
out.writeObject(new Person(12, "张三"));

ObjectInputStream in = new ObjectInputStream(new FileInputStream("/Users/a.txt"));
Person p = (Person) in.readObject();
System.out.println(p.getId());
System.out.println(p.getName());
~~~

### 支持格式化打印数据的装饰器类

PrintStream和PrintWriter可以将数据按照一定的格式转化为字符串后，再写入到输出流。前面讲到System.out、System.err就是PrintStream类型的。示例代码如下所示。
~~~java
PrintStream printStream =new PrintStream(new FileOutputStream("/Users/a.txt"));
printStream.print(124); //int->Integer->toString(), 写入字符串"124"
printStream.printf("hello %d", 43); //写入字符串"hello 43"
~~~

除了以上装饰器类之外，InputStreamReader、OutputStreamWriter这两个原始类从功能上很像装饰器类。InputStreamReader可以充当InputStream的装饰器类，OutputStreamWriter可以充当OutputStream的装饰器类。它们可以将字节流转化为字符流。示例代码如下所示。从这一点上，我们也可以看出，java.io类库的设计有很多不合理的地方，而更晚开发的java.nio类库在设计上明显要合理很多，下面会详细讲到。
~~~java
OutputStream outStream = new FileOutputStream("/Users/a.txt");
OutputStreamWriter writer = new OutputStreamWriter(outStream, "gbk");
writer.write("王a争"); //按照gbk编码将字符串写入文件
~~~

### 其他装饰器类
比如PushbackInputStream、PushbackReader、SequenceInputStream、LineNumberReader，因为使用的比较少，可以自行查阅。

## java.nio类库
nio的全称为New I/O。因为相对于java.io，java.nio新增了非阻塞的I/O访问方式，所以，有人把nio解读为Non-blocking I/O。
尽管java.nio从功能上可以完全替代java.io，但是，在平时的开发中，对于普通的文件读写，我们更倾向于使用简单的java.io，而对于比较复杂的网络编程，我们更倾向于使用java.nio。基于此，还有人把nio解读为Network I/O。

Stream是java.io类库中的一个核心概念。所有的I/O都抽象为Stream。读写Stream就等同于读写I/O。
而在java.nio中，Stream被Channel所替代。除此之外，java.nio还引入了一个新的概念：Buffer，用来存储待写入或读取的数据。接下来，我们先拿一个比较简单的文件读写的例子，来看一下Channel和Buffer是如何使用的。示例代码如下所示。
~~~java
FileChannel channel = FileChannel.open(Paths.get("/Users/wangzheng/in.txt"));
ByteBuffer buffer = ByteBuffer.allocate(512);
while (channel.read(buffer) != -1) {
  // 处理buffer中的数据data
}
~~~

除了上面提到的Buffer、Channel之外，java.nio中还有两个重要的概念：Selector和异步Channel，接下来详细介绍一下java.nio中的这4个核心概念。

### Buffer
Buffer本质上就是一块内存，相当于我们在使用java.io编程时申请的byte数组。常用到Buffer有ByteBuffer、CharBuffer、ShortBuffer、IntBuffer、LongBuffer、FloatBuffer、DoubleBuffer、MappedByteBuffer。这些Buffer的不同之处在于解析数据的方式不同，比如CharBuffer按照字符来解析数据，有点类似java.io中的字符流。

上面讲到，java.io类库的设计存在问题。java.io类库分别为字符流和字节流设计了不同的类，在代码实现上有些重复。毕竟字符流和字节流的I/O读写操作都是相同的，唯一的区别只是数据的解析方式不同。字节流类按照字节解析数据。字符流类按照字符解析数据。
java.nio对此做了优化，将解析这部分功能抽取出来，独立到Buffer类中。不同的Channel跟不同的Buffer组合在一起，可以实现不同的IO读写需求。比如，FileChannel跟ByteBuffer组合在一起，就相当于java.io中的文件字节流类（FileInputStream、FileOutputStream），FileChannel跟CharBuffer组合在一起，就相当于java.io中的文件字符流类（FileReader、FileWriter）。

实际上，Channel和Buffer独立开发，然后组合起来使用，这种设计思路应用的就是面向对象设计中的“组合优于继承”的设计思想。通过组合来替代继承，避免了继承带来的组合爆炸问题。正因如此，在实现相同甚至更多功能的情况下，java.nio中类的个数却比java.io中类的个数少。

### Channel
常用的Channel有FileChannel、DatagramChannel、SocketChannel、ServerSocketChannel。FileChannel用于文件读写。DatagramChannel、SocketChannel、ServerSocketChannel用于网络编程。DatagramChannel用来读写UDP数据，SocketChannel和ServerSocketChannel用来读写TCP数据。SocketChannel和ServerSocketChannel的区别在于，ServerSocketChannel用于服务器编程，可以使用accept()函数监听客户端（SocketChannel）的连接请求。

java.io中的Stream要么只能读，要么只能写，而java.nio中的Channel既可以读，也可以写。
这也是java.nio比java.io类少的另一个重要原因。除此之外，Channel的设计也利用了“组合优于继承”的设计思想。 java.nio中包含大量的Channel接口，每个接口定义了一种功能。每个Channel类通过实现不同的接口组合，来支持不同的功能组合。

Channel有两种运行模式：阻塞模式和非阻塞模式。其中，FileChannel只支持阻塞模式。DatagramChannel、SocketChannel、ServerSocketChannel支持阻塞和非阻塞两种模式，默认为阻塞模式。我们可以调用configureBlocking(false)函数将其设置为非阻塞模式。非阻塞Channel一般会配合Selector，用于实现多路复用I/O模型。

那么，到底什么是阻塞模式？什么是非阻塞模式呢？
线程在调用read()或write()函数对I/O进行读写时，如果I/O不可读或者不可写（待会解释这两个的意思），那么，在阻塞模式下，read()或write()函数会等待，直到读取到数据或者写入完成时才会返回，在非阻塞模式下，read()或write()函数会直接返回，并报告读取或写入未成功。

在操作系统层面，主要的I/O有：文件、网络、标准输入输出、管道。文件是没有非阻塞模式的。毕竟文件不存在不可读和不可写的情况。网络、标准输入输出、管道都存在阻塞和非阻塞两种模式。我们拿最常用的网络来举例讲解。
一般来讲，应用程序调用read()或write()函数读取或写入数据，数据会在应用程序缓冲区、内核缓冲区、I/O设备这三者之间拷贝传递。

当调用read()函数读取数据时，如果内核读缓冲区中没有可读数据，比如网络连接的对方此时并未发送数据过来，那么，在阻塞模式下，read()函数会等待，直到对方发送数据过来，内核读缓冲区中有数据可读时，才会将内核读缓冲区中的数据拷贝到应用程序缓存中，然后read()函数才返回，在非阻塞模式下，read()函数会直接返回并报告读取情况。

当调用write()函数时，如果内核写缓冲区中没有足够空间承载应用程序缓存中的数据，比如网络不好，原来的数据还没来得及发送出去，那么，在阻塞模式下，write()函数会等待，直到内核写缓冲区中有足够空间，并且应用程序缓冲区中的数据全部写入内核写缓冲区，write()函数才会返回。在非阻塞模式下，write()函数会能写多少写多少，即便还有一部分未能写入内核写缓冲区，也不会等待，直接返回并报告写入情况。

实际上，除了read()和write()函数有阻塞和非阻塞这两种模式之外，ServerSocketChannel中用于服务器接收客户端连接的accpet()函数，也有阻塞和非阻塞两种模式。在阻塞模式下，调用accept()函数会等待，直到有客户端连接到来才返回。在非阻塞模式下，如果没有客户端连接到来，调用accept()函数会直接返回。

### Selector
在网络编程中，如果我们使用非阻塞模式的read()、write()、accept()函数，那么，我们需要通过while循环，不停轮询调用read()、write()、accept()函数，查看是否有数据可读、是否可写、是否有客户端连接到来。示例代码如下所示。
~~~java
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.bind(new InetSocketAddress("127.0.0.1", 1192));
serverChannel.configureBlocking(false);
ByteBuffer buffer = ByteBuffer.allocate(1024);

SocketChannel clientChannel = null;
while (clientChannel == null) {
  clientChannel = serverChannel.accept();
}

while (clientChannel.read(buffer) == -1);

buffer.flip(); //将buffer从"用于读"变成"用于写"
while(buffer.hasRemaining()) {
  clientChannel.write(buffer); // echo,读了啥就写啥
}
~~~

上述代码充斥着while轮询，显然不够优雅。多路复用I/O模型便用来解决这个问题。

多路复用I/O模型是网络编程中非常经典的一种I/O模型。为了实现多路复用I/O模型，Unix提供了epoll库，Windows提供了iocp库，BSD提供了kequeue库...Java作为一种跨平台语言，对不同操作系统的实现方式进行了封装，提供了统一的Selector。

我们可以调用register()函数，将需要监听的Channel注册到Selector中。Selector底层会通过轮询的方式，查看哪些Channel可读、可写、可连接，并将其返回处理。关于Selector的使用示例代码，在后面会给出。

### 异步Channel
尽管使用Selector可以避免程序员自己手写轮询代码，但是，Selector底层仍然依赖轮询来实现。在JDK7中，java.nio类库做了升级，引入了支持异步模式的Channel，主要包括：AsynchronousFileChannel、AsynchronousSocketChannel、AsynchronousServerSocketChannel。

前面讲到的Channel都是同步模式的。那么，什么是同步模式？什么是异步模式呢？同步和异步这两个概念，跟阻塞和非阻塞又是否有联系呢？

我们通过一个生活中的例子来形象解释一下。假设你去一家餐厅就餐，因为就餐的人太多，需要取号等位。取号之后，如果你站在餐厅门口一直等待被叫号，啥都不干，那么，这就是阻塞模式。如果你先去商场里逛一逛，一会回来看一下有没有轮到你，没有就继续再去逛，那么，这就是非阻塞模式。
如果你在取号时，登记了手机号码，那么，你就可以放心去逛商场了。等叫到你的号时，服务员会打电话通知你，这就是异步模式。相反，如果需要自己去查看有没有轮到你，不管是阻塞模式还是非阻塞模式，都是同步模式。

在异步模式下，Channel不再注册到Selector，而是注册到操作系统内核中，由内核来通知某个Channel可读、可写或可连接，java.nio收到通知之后，为了不阻塞主线程，会使用线程池去执行事先注册的回调函数。关于异步Channel的使用示例代码，在后面会给出。

## Java IO模型
Java中常被提及的I/O模型有三个：阻塞I/O模型（BIO）、非阻塞I/O模型（NIO）、异步I/O模型（AIO）。我们依次看下这3种常见的I/O模型。

### 阻塞I/O模型（BIO）
前面讲过，I/O访问模式有两种：阻塞模式和非阻塞模式。阻塞I/O模型指的是利用阻塞模式来实现服务器。一般来说，这种模型需要配合多线程来实现。

一般来讲，服务器需要连接大量客户端，因为read()函数是阻塞函数，所以，为了实时接收客户端发来的数据，服务器需要创建大量线程，每个线程负责等待读取（调用read()函数）一个客户端的数据。因为java.io支持阻塞模式，java.nio既支持阻塞模式又支持非阻塞模式，所以，java.io和java.nio都可以实现阻塞I/O模型。我们使用java.io来编写示例代码，如下所示。注意，使用java.io进行网络编程，需要配合java.net类库。java.net类库用于管理连接，java.io用于读写数据。在下面代码中，Socket、ServerSocket都是java.net包中的类。

如果有n个客户端连接服务器，那么，服务器需要创建n+1个线程，其中n个线程用于调用read()函数，另外一个线程用来调用accept()阻塞函数。当连接的客户端非常多时，服务器需要创建大量线程，而每个线程会分配一个线程栈，需要占用一定的内存空间。当线程比较多时，内存资源的消耗就会比较大。并且大量线程来回切换，也会导致服务器整体处理性能的下降。除此之外，大部分线程可能都阻塞在read()函数上，等待数据的到来，什么都不做但又要白白占用内存和线程资源，非常浪费。
~~~java
public class BioEchoServer {
  public static void main(String[] args) throws IOException {
    ServerSocket serverSocket = new ServerSocket();
    serverSocket.bind(new InetSocketAddress("127.0.0.1", 1192));
    while (true) {
      // accept()为阻塞函数，直到有连接到来才返回
      Socket clientSocket = serverSocket.accept();
      // 为了每个客户端单独创建一个线程处理
      new Thread(new ClientHandler(clientSocket)).start();
    }
  }

  private static class ClientHandler implements Runnable {
    private Socket socket;
    public ClientHandler(Socket socket) {
      this.socket = socket;
    }

    @Override
    public void run() {
      byte[] data = new byte[1024];
      while (true) { //持续接收客户端发来的数据
        try {
          // read()为阻塞函数，直到读取到数据再返回
          socket.getInputStream().read(data);
          // write()为阻塞函数，全部写完成才会返回
          socket.getOutputStream().write(data); //echo
        } catch (IOException e) {
          // log and exit
          break;
        }
      }
    }
  }
}
~~~


### 非阻塞I/O模型（NIO）
非阻塞I/O模型指的是利用非阻塞模式来开发服务器，一般需要配合Selector多路复用器。因此，这种模型也叫做多路复用I/O模型。因为java.io只支持阻塞模式，所以，这种模型只能通过java.nio来实现。非阻塞I/O模型的示例代码如下所示。利用java.nio进行网络编程，也像java.io那样，需要java.net类库的配合，比如代码中的InetSocketAddress就是java.net中的类。
~~~java
public class NioEchoServer {
  public static void main(String[] args) throws IOException {
    // Selector
    Selector selector = Selector.open();

    // create serverChannel and register to selector
    ServerSocketChannel serverChannel = ServerSocketChannel.open();
    serverChannel.bind(new InetSocketAddress("127.0.0.1", 1192));
    serverChannel.configureBlocking(false);
    serverChannel.register(selector, SelectionKey.OP_ACCEPT);

    ByteBuffer buffer = ByteBuffer.allocate(1024);
    while (true) {
      int channelCount = selector.select();
      if (channelCount > 0) {
        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()) {
          SelectionKey key = iterator.next();
          if (key.isAcceptable()) {
            // create clientChannel and register to selector
            SocketChannel clientChannel = serverChannel.accept();
            clientChannel.configureBlocking(false);
            clientChannel.register(selector, SelectionKey.OP_READ);
          } else if (key.isReadable()) {
            SocketChannel clientChannel = (SocketChannel) key.channel();
            clientChannel.read(buffer);
            buffer.flip(); //从"用于读"变为"用于写"
            while (buffer.hasRemaining()) { //也可以注册到selector中
              clientChannel.write(buffer); //echo
            }
            buffer.clear(); //重复利用
          }
        }
      }
    }
  }
}
~~~

如上代码所示，大部分情况下，我们都不需要监听Channel是否可写，毕竟网络写入跟文件写入类似，大部分情况下都不需要等待。只有当写入出现问题时，比如write()函数返回0，表示网络拥塞，此时才需要如下代码所示，将Channel注册到Selector中，等待可写。
~~~java
clientChannel.register(selector, SelectionKey.OP_WRITE);
~~~

如果有n个客户端连接服务器，那么，就会创建n+1个Channel，其中一个serverChannel用于接受客户端的连接，另外n个clientChannel用于与客户端进行通信。这n+1个Channel均注册到Selector中。Selector会间隔一定时间轮训这n+1个Channel，查找可连接、可读、可写的Channel，然后再进行连接、读取、写入操作。

需要注意的是，并不是所有的Channel都可以注册到Selector中被监听，只有实现了SelectableChannel接口的Channel才可以，比如DatagramChannel、SocketChannel、ServerSocketChannel。因为FileChannel没有实现SelectableChannel，并且不支持非阻塞模式，所以，无法被Selector监听。

多路复用I/O模型只需要一个线程即可，解决了阻塞I/O模型线程开销大的问题。不过，这种模型依然存在问题。如果某些clientChannel读写的数据量比较大，或者逻辑操作比较复杂，耗时比较久，因为所有的工作都在一个线程中完成，那么其他clientChannel便迟迟得不到处理，最终的效果就是，服务器响应客户端的延迟很大。


### 异步I/O模型（AIO）
为了解决非阻塞I/O模型所有的工作都在一个线程中完成，那么其他clientChannel便迟迟得不到处理，最终的效果就是，服务器响应客户端的延迟很大，可以引入线程池，对于Selector检测到有数据可读的clientChannel，我们从线程池中取线程来处理。阻塞I/O模型也用到了多线程，跟这里的区别在于，不管有没有数据可读，阻塞I/O模型中的每个clientSocket都会一直占用线程。而这里的多线程只会处理经过Selector筛选之后有可读数据的clientChannel，并且处理完之后就释放回线程池，线程的利用率更高。

而上述多路复用加线程池所实现的I/O模型就是异步I/O模型。
对于异步I/O模型，使用java.nio的异步Channel来实现更加优雅。示例如下代码所示。如果我们通过异步Channel调用accept()、read()、write()函数，那么，当有连接建立、数据读取完成或数据写入完成时，底层会通过线程池执行对应的回调函数。
~~~java
public class AioEchoServer {
  public static void main(String[] args) throws IOException, InterruptedException {
    AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel.open();
    serverChannel.bind(new InetSocketAddress("127.0.0.1", 1192));
    // 异步accept() ，设置当有下一个客户端连接时由 AcceptCompletionHandler 进行异步回调处理连接
    // 第一个参数是需要传递给异步回调函数的数据
    serverChannel.accept(null, new AcceptCompletionHandler(serverChannel));
    // 保持主线程运行，防止程序退出
    Thread.sleep(Integer.MAX_VALUE);
  }

  // AcceptCompletionHandler 实现了CompletionHandler接口，用于处理客户端的连接请求
  private static class AcceptCompletionHandler
      implements CompletionHandler<AsynchronousSocketChannel, Object> {
    private AsynchronousServerSocketChannel serverChannel;
    public AcceptCompletionHandler(AsynchronousServerSocketChannel serverChannel) {
      this.serverChannel = serverChannel; 
    }
    
    // 当有新的客户端连接时，completed 方法被异步调用
    // 第一个参数是客户端连接，第二个参数是设置回调时设置传递的参数
    @Override
    public void completed(AsynchronousSocketChannel clientChannel, Object attachment) {
      // 继续异步accept() ，设置当有下一个客户端连接时由 AcceptCompletionHandler 进行异步回调处理连接
      serverChannel.accept(attachment, this);
      ByteBuffer buffer = ByteBuffer.allocate(1024);
      // 异步read()，设置当有下一个客户端连接时由 ReadCompletionHandler 进行异步回调处理读取的数据
      // 第一个参数是目标缓冲区，异步读取的数据存储在这个缓冲区中
      // 第二个参数是传递给异步回调函数的数据，这里是将读取的数据交给异步回调函数处理
      clientChannel.read(buffer, buffer, new ReadCompletionHandler(clientChannel)); 
    }

    @Override
    public void failed(Throwable exc, Object attachment) {
      // 当接收客户端连接失败时调用此方法。这里可以添加日志记录
    }
  }

  // ReadCompletionHandler 实现了CompletionHandler接口，用于处理从客户端的连接读取的数据
  private static class ReadCompletionHandler 
      implements CompletionHandler<Integer, ByteBuffer> {
    private AsynchronousSocketChannel clientChannel;
    public ReadCompletionHandler(AsynchronousSocketChannel clientChannel) {
      this.clientChannel = clientChannel;
    }
    
    @Override
    public void completed(Integer result, ByteBuffer buffer) {
      // 从"用于读"变为"用于写"   
      buffer.flip();
      // 异步write()。回调函数为null，写入完成就不用回调了
      clientChannel.write(buffer, null, null); // echo
    }

    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
      // 读取数据失败时调用此方法。这里同样可以添加日志记录
    }
  }
}
~~~

实际上，在平时的开发中，我们一般不会直接使用java.nio类库，而是使用Netty等框架来进行网络编程，这些框架封装了网络编程的复杂性，使用起来更加简单，开发效率更高。除了以上三种常见的I/O模型之外，实际上，还有更多更加复杂的I/O模型，比如Netty框架提供的Reactor模型。关于Netty等网络编程知识，就不深入讲解了。

你可能还听过其他I/O模型的分类，比如在《Unix网络编程》一书中，介绍了Unix操作系统的5种I/O模型：阻塞I/O模型、非阻塞I/O模型、多路复用I/O模型、信号驱动I/O模型、异步I/O模型。那么，Unix操作系统下的I/O模型跟Java I/O模型有什么联系呢？

实际上，不同的操作系统会提供不同的I/O模型。Java是一种跨平台语言，为了屏蔽各个操作系统I/O模型的差异，设计了3种新的I/O模型：BIO（阻塞I/O）、NIO（非阻塞I/O）、AIO（异步I/O），并且提供了I/O类库来支持这3种I/O模型的代码实现。而Java的I/O类库底层需要依赖操作系统的I/O接口来实现，因此，从本质上来讲，Java I/O模型只是对操作系统I/O模型的重新封装。

## 对比java.io与java.nio
从功能上来看，java.nio完全可以替代java.io，那么，在平时开发中，我们是不是应该首选java.nio呢？在新开发的项目中，是不是就不应该使用老的java.io呢？

实际上，在某些情况下，我们确实必须使用java.nio，比如网络编程。尽管使用java.io，并配合java.net，也可以进行网络编程，但java.io只支持阻塞模式，只能实现阻塞I/O模型，对于大部分网络编程来说，都是不够的。而java.nio提供了非阻塞模式、Selector多路复用器、异步模式，能够实现更加高性能的网络模型，比如非阻塞I/O模型、异步I/O模型。相比java.io而言，在网络编程方面，java.nio的优势更加明显。

但是，在某些情况下，到底使用java.io还是java.nio，会有一些争论，比如文件读写。前面提到，文件读写只支持阻塞模式，因此，使用java.io或java.nio都可以。有些人认为，使用java.io进行文件读写，代码编写更加简单。有些人则认为，java.nio的文件读写功能更加丰富。我个人认为，既然有争论，就说明两者没有哪个更有绝对优势，不然也就不会有争论了。因此，对于使用java.io还是java.nio进行文件读写，按照你的喜好或者团队的编程习惯来选择就好。总结一下的话，对于网络编程，我们首选java.nio，对于文件读写，java.io和java.nio都可以。


# 一个简单的http服务器
~~~java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.util.HashMap;
import java.util.Map;
import java.util.function.BiConsumer;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class GatewayHandler {
    
    // 路由表，将路径映射到处理函数
    private static final Map<String, BiConsumer<AsynchronousSocketChannel, String>> routes = new HashMap<>();
    
    // Jackson库 用于JSON解析
    // <dependency>
    //     <groupId>com.fasterxml.jackson.core</groupId>
    //     <artifactId>jackson-databind</artifactId>
    //     <version>2.14.1</version>
    // </dependency>
    private static final ObjectMapper objectMapper = new ObjectMapper(); 

    public static void main(String[] args) throws IOException {
        // 注册路由
        registerRoutes();

        // 创建一个自定义的线程池
        ExecutorService threadPool = Executors.newFixedThreadPool(10); // 固定大小的线程池
        // 创建一个AsynchronousChannelGroup，绑定自定义线程池
        AsynchronousChannelGroup channelGroup = AsynchronousChannelGroup.withThreadPool(threadPool);

        AsynchronousServerSocketChannel serverChannel = AsynchronousServerSocketChannel.open(channelGroup);
        serverChannel.bind(new InetSocketAddress("127.0.0.1", 8080));
        System.out.println("HTTP server started at http://127.0.0.1:8080/");

        serverChannel.accept(null, new AcceptCompletionHandler(serverChannel));

        // 保持主线程运行
        try {
            Thread.currentThread().join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 注册路由
    private static void registerRoutes() {
        routes.put("/hello", (clientChannel, request) -> {
            sendResponse(clientChannel, "200 OK", "Hello, World!");
        });

        routes.put("/goodbye", (clientChannel, request) -> {
            sendResponse(clientChannel, "200 OK", "Goodbye, World!");
        });

        routes.put("/submit", (clientChannel, request) -> {
            // 解析并处理JSON请求体
            JsonNode jsonRequest = parseJsonRequestBody(request);
            if (jsonRequest != null) {
                String responseContent = "Data received: " + jsonRequest.toString();
                sendResponse(clientChannel, "200 OK", responseContent);
            } else {
                sendResponse(clientChannel, "400 Bad Request", "Invalid JSON format.");
            }
        });
    }

    // 发送HTTP响应
    private static void sendResponse(AsynchronousSocketChannel clientChannel, String status, String content) {
        String httpResponse = "HTTP/1.1 " + status + "\r\n" +
                "Content-Length: " + content.length() + "\r\n" +
                "Content-Type: application/json\r\n" +
                "\r\n" +
                content;

        ByteBuffer responseBuffer = ByteBuffer.wrap(httpResponse.getBytes(StandardCharsets.UTF_8));
        clientChannel.write(responseBuffer, responseBuffer, new WriteCompletionHandler(clientChannel));
    }

    // 解析JSON请求体
    private static JsonNode parseJsonRequestBody(String request) {
        try {
            String requestBody = extractRequestBody(request);
            return objectMapper.readTree(requestBody); // 使用Jackson解析JSON
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        }
    }

    // 从HTTP请求中提取请求体（这里假设请求体是简单的文本格式）
    private static String extractRequestBody(String request) {
        String[] requestLines = request.split("\r\n");
        StringBuilder requestBody = new StringBuilder();
        boolean isBody = false;
        for (String line : requestLines) {
            if (line.isEmpty()) {
                isBody = true; // 空行之后就是请求体
            } else if (isBody) {
                requestBody.append(line);
            }
        }
        return requestBody.toString();
    }

    // 内部类：处理接受客户端连接
    private static class AcceptCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, Void> {
        private final AsynchronousServerSocketChannel serverChannel;

        public AcceptCompletionHandler(AsynchronousServerSocketChannel serverChannel) {
            this.serverChannel = serverChannel;
        }

        @Override
        public void completed(AsynchronousSocketChannel clientChannel, Void attachment) {
            // 继续接受其他客户端连接
            serverChannel.accept(null, this);

            // 读取客户端请求
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            clientChannel.read(buffer, buffer, new ReadCompletionHandler(clientChannel));
        }

        @Override
        public void failed(Throwable exc, Void attachment) {
            exc.printStackTrace();
        }
    }

    // 内部类：处理读取客户端请求
    private static class ReadCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel clientChannel;

        public ReadCompletionHandler(AsynchronousSocketChannel clientChannel) {
            this.clientChannel = clientChannel;
        }

        @Override
        public void completed(Integer result, ByteBuffer buffer) {
            buffer.flip();
            String request = StandardCharsets.UTF_8.decode(buffer).toString();
            System.out.println("Received request: \n" + request);

            // 简单地解析HTTP请求的第一行
            String[] requestLines = request.split("\r\n");
            String[] requestLine = requestLines[0].split(" ");
            String method = requestLine[0];
            String path = requestLine[1];

            // 从路由表中查找处理函数
            BiConsumer<AsynchronousSocketChannel, String> handler = routes.get(path);
            if (handler != null) {
                handler.accept(clientChannel, request);
            } else {
                sendResponse(clientChannel, "404 Not Found", "The requested resource was not found.");
            }
        }

        @Override
        public void failed(Throwable exc, ByteBuffer buffer) {
            exc.printStackTrace();
            try {
                clientChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    // 内部类：处理写入响应数据到客户端
    private static class WriteCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {
        private final AsynchronousSocketChannel clientChannel;

        public WriteCompletionHandler(AsynchronousSocketChannel clientChannel) {
            this.clientChannel = clientChannel;
        }

        @Override
        public void completed(Integer result, ByteBuffer buffer) {
            if (buffer.hasRemaining()) {
                clientChannel.write(buffer, buffer, this);
            } else {
                try {
                    clientChannel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void failed(Throwable exc, ByteBuffer buffer) {
            exc.printStackTrace();
            try {
                clientChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
~~~

# 操作系统的用户态和内核态
对于软件开发来说，分层是一种非常常见的设计思路。操作系统作为一种特殊的软件，当然也不例外。我们拿Linux操作系统举例讲解。

操作系统包含计算机运行所需要的核心程序，这部分程序用来访问硬件资源，比如调度CPU、读写磁盘、网卡、内存等。我们把这部分程序叫做操作系统内核（简称内核）。因为操作硬件资源非常容易出错，并且一旦出错，错误将非常严重，大部分情况下都会导致计算机宕机，所以，操作系统不允许应用程序直接访问硬件资源。如果应用程序需要访问硬件资源，比如读写磁盘，那么，只能通过操作系统提供的API来实现。我们把操作系统提供给应用程序使用的API称为系统调用。

系统调用比较底层，使用起来不够方便，于是，Linux操作系统在此之上又提供了库函数，比如Glibc库、Posix库，对系统调用进行封装，提供更加简单易用的函数，供应用程序开发使用，比如Glibc中的malloc()函数底层封装了sbrk()系统调用，fread()、fwrite()函数底层封装了read()、write()系统调用。在进行应用程序开发时，我们既可以使用库函数，也可以直接使用系统调用，比如，对于内存分配，我们一般使用malloc()库函数，对于文件读写，我们一般直接使用read()、write()系统调用。

除此之外，Linux操作系统还提供了Shell这一特别的程序，也就是我们平时所说的命令行。Shell让我们能够在不进行编程的情况下，通过在命令行中运行Shell命令或脚本，达到访问硬件的目的，比如，使用cp命令拷贝文件，使用rm命令删除文件等。

为了避免应用程序在运行时，访问到内核所用的内存空间，操作系统将虚拟内存空间分为内核空间和用户空间两部分。而我们经常提到的内核态和用户态，实际上指的是CPU运行所处的状态。当CPU执行内核程序时，CPU进入内核态。在内核态下，CPU拥有最高权限，可以执行所有的机器指令，当然，也可以访问硬件设备。当CPU执行应用程序时，CPU进入用户态，在用户态下，CPU权限被限制，只能执行部分机器指令，当然，也无法访问硬件设备。除此之外，在内核态下，CPU可以访问所有的虚拟内存空间，包括用户空间和内核空间，在用户态下，CPU只能访问用户空间，不能访问内核空间。

## 系统调用与上下文切换
当应用程序调用操作系统的系统调用时，CPU从用户态切换到内核态，当系统调用执行完成之后，CPU又从内核态切换到用户态。我们把这种状态的切换称为上下文切换。实际上，上下文切换是一个比较宽泛的概念，在很多场景中的都会用到，比如线程切换也会引起上下文切换。

对于普通函数调用来说，应用程序只需要将局部变量、参数、返回地址等信息存入函数调用栈，并同步保存和修改一些寄存器即可，比如SP、BP寄存器，函数调用产生的额外耗时相对较少。尽管系统调用从本质上也是一种函数调用，但相比于应用程序内的普通函数调用来说，系统调用要慢很多，耗时的地方主要在于内核态与用户态的上下文切换 ，详细来讲，主要有以下两方面。

### 寄存器保存与恢复耗时
因为操作系统内核是不信赖应用程序的，所以，操作系统内核不会使用应用程序开辟的位于用户空间的函数调用栈。操作系统会在内核空间中分配新的函数调用栈，供内核程序执行的过程中使用。当调用系统调用时，从用户空间的函数调用栈切换到内核空间的函数调用栈，使得操作系统需要更新更多跟栈相关的寄存器，比如SS栈基址寄存器。除此之外，因为应用程序的代码和内核程序的代码所存储的位置也不同，所以，当从应用程序的代码切换到内核代码时，CS代码段基址寄存器也需要更新。并且，在更新之前，操作系统需要将这些寄存器原来的值保存下来，以便重新切换回用户态之后恢复执行。

### 缓存失效带来的性能损耗
除此之外，根据局部性原理，CPU有L1、L2、L3三级Cache，用于缓存将要执行的代码以及所需的内存数据。应用程序的代码以及所需的数据存储在用户空间，内核代码以及所需的数据存储在内核空间，显然不是相邻的。因此，用户态到内核态的转化会导致CPU缓存失效。

## I/O读写的底层实现原理
尽管Java I/O类库（java.io和java.nio）使用起来比较简单，但其底层实现原理却比较复杂。Java作为一个跨平台的语言，为了屏蔽操作系统的差异，提供了统一的Java I/O类库。在不同操作系统下，Java I/O类库中的函数底层调用不同的系统调用和库函数来实现。在Linux操作系统下，Java I/O类库中的read()、write()函数，底层通过调用Linux操作系统的read()、write()系统调用来实现。因此，使用Java中的read()、write()函数进行I/O读写，势必会涉及用户态和内核态的切换。I/O读写过程中涉及到的一些关键步骤如下

### 1. open()
在进行I/O读写之前，操作系统需要先建立与I/O设备的连接。在Linux操作系统下，一切皆文件，I/O设备也不例外。Linux操作系统会为每个与I/O设备建立的连接，分配一个文件描述符（file descriptor）。对应到代码层面，当我们调用open()系统调用建立连接时，open()系统调用会将连接对应的文件描述符作为函数返回值返回。后续对文件描述符的读写就等价于对I/O设备的读写。

操作系统为每个文件描述符都分配了一个读缓冲区和一个写缓冲区，分别对应到内核读缓冲区和内核写缓冲区。内核读写缓冲区只有在第一次被使用（调用read()或write()系统调用）时，才会真正被分配内存空间。默认读缓冲区的大小一般为8192字节，写缓冲区的大小一般为16384字节，当然，我们也可以根据业务需求，重新设置内核读写缓冲区的大小。

### 2. close()
因为文件描述会附带一些信息存储在内存中，并且每个文件描述符都分配有内核缓冲区，会占用一定的存储空间，所以，在读写完成之后，应用程序需要调用close()系统调用，释放文件描述符及其内核缓冲区。

### 3. read()
当应用程序调用Java I/O类库中的read()函数时，read()函数会调用操作系统的read()系统调用.操作系统会先检查内核读缓冲区中有没有足够的数据，如果有足够数据，那么就直接将数据从内核读缓冲区拷贝到应用程序缓冲区。否则，操作系统将从磁盘中读取数据，拷贝到内核读缓冲区中，然后再拷贝到应用程序缓冲区，之后read()函数便可返回。

### 4. write()
当应用程序调用Java I/O类库中的write()函数时，write()函数会调用操作系统的write()系统调用，操作系统会先将应用程序缓冲区中的数据，拷贝到内核写缓冲区，这个时候，对于应用程序来说，写操作就完成了，write()函数便返回。

操作系统会根据一定的规则，比如写缓冲满了或到达了一定时间间隔，在某个时刻，将写缓冲区中的数据，一并写入I/O设备。如果我们希望write()返回之后，数据立刻存储到磁盘，那么需要显式地调用强制落盘函数，比如使用Linux操作系统下的sync()系统调用。当然，如果我们调用了close()系统调用，就不需要再调用sync()系统调用了，这是因为调用close()系统调用会释放内核缓冲区，为了防止数据丢失，会默认自动调动sync()系统调用，将内核缓冲区中的数据写入I/O设备。

对于上述读写流程，我们拿文件读写举例，示例代码如下所示。代码中的buffer就是应用程序缓冲区。
~~~c
// Linux下读写文件的C语言代码实现
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int main (int argc, char *argv[]) {
    int rfd;
    int wfd;
    int nbytes;
    char rfile[] = "/users/root/in.txt";
    char wfile[] = "/users/root/out.txt";
    char buffer[256]; //应用程序缓冲区
    rfd = open(rfile, O_RDONLY, 0666);
    wfd = open(wfile, O_CREAT | O_WRONLY, 0666);
    if(rfd < 0 || wfd < 0){
        printf("open file failed!\n");
        return -1;
    }
    while((nbytes = read(rfd, buf, 255)) > 0){
        write(wfd, buffer, nbytes);
    }
    close(rfd);
    close(wfd);
    return 0;
}
~~~
在上述读写流程中，读写数据均需要进行2次数据拷贝。我们拿读数据来举例。当调用read()函数读取数据时，数据从I/O设备拷贝到内核缓冲区，再从内核缓冲区拷贝到应用程序缓冲区，总共发生了2次数据拷贝。既然操作系统内核既可以访问内核空间，又可以访问用户空间，那么，它为什么不直接将数据从I/O设备拷贝到应用程序缓冲区呢？这样做不就能减少了一次数据拷贝吗？

内核缓冲区用于缓存从I/O设备中读取的数据，进而减少操作系统与I/O设备的交互次数。应用程序缓冲区是应用程序维护的，应用程序对其有主宰权，内核代码无法控制其大小和生命周期，有很多内核不可控的不稳定因素。出于稳妥起见，操作系统为I/O设备在内核空间申请了内核缓冲区而没有复用应用程序缓冲区。

那么问题又来了，既然已经有了内核缓冲区为I/O设备提供数据缓存功能，那么，为什么Java I/O类库还提供支持缓存功能的BufferedInputStream、BufferedOutputStream类呢？
我们拿BufferedInputStream举例讲解。BufferedInputStream相当于在用户空间又增加一层缓存。当调用read()函数读取数据时，应用程序会先从位于用户空间的缓存中读取，如果缓存中无数据可读，这时才会调用Linux操作系统的read()系统调用，也就是说，增加了一层用户空间的缓存，可以减少调用系统调用的次数。我们知道，系统调用会导致上下文切换，上下文切换是比较耗时的，所以，减少系统调用也会提高read()函数的性能。

## CPU减负神器之DMA技术
从I/O读写的底层实现原理，我们可以发现，在I/O读写过程中，CPU一直参与其中，负责I/O设备与内核缓冲区之间，以及内核缓冲区与应用程序缓冲区之间的数据拷贝。而CPU最擅长的是运算，比如加法运算、位运算，让CPU去做拷贝数据这种简单工作（将二进制位0或1从一个存储单元移动到另一个存储单元），实际上是大材小用。除此之外，相对于CPU来说，像硬盘、网卡等I/O设备的读写速度非常慢，在I/O读写的过程中，CPU会一直被占用，无法处理其他事情，这无疑是非常浪费CPU资源的。

于是，科学家们便发明了DMA（Direct Memory Access）技术。通过在主板上安装一个叫做DMAC（DMA Controller，DMA控制器）的协处理器（或叫芯片），协助CPU来完成I/O设备的数据读写工作。随着计算机的发展，安装在计算机上的I/O设备越来越多，仅在主板上安装一个通用的DMAC已经远远不够了，因此，现在很多I/O设备都自带DMAC，比如硬盘、网卡、显示器都有各自的DMAC。

### 具体来讲，DMA是怎样工作的呢？
当调用read()函数从I/O设备读取数据时，通过系统调用，CPU会进入内核态，发送I/O请求到DMAC，告知DMAC从I/O设备中读取哪些数据到哪块内存，之后CPU便去做其他事情，由DMAC来完成将数据从I/O设备拷贝到内核缓冲区的工作。当DMAC完成拷贝工作之后，DMAC通过中断，通知CPU内核缓冲区中的数据已经准备就绪，然后CPU再将内核读缓冲区中的数据拷贝到应用程序缓冲区。

当调用write()函数将数据写入I/O设备时，通过系统调用，CPU会进入内核态，将数据从应用程序缓冲区拷贝到内核缓冲区，然后发送I/O请求给DMAC，告知其将哪块内存中的数据拷贝到I/O设备中，之后CPU便去做其他事情了，由DMAC来完成将数据从内核写缓冲区中拷贝到I/O设备中的工作。

通过DMA技术，不管是读取I/O数据还是写入I/O数据，CPU只需要参与一次用户态和内核态之间的数据拷贝，硬件I/O操作和内核态之间的数据拷贝交给DMA操作，CPU利用率提高。

不过，你可能会说，DMA技术只不过是对CPU“减负”而已，由CPU拷贝换成了DMA拷贝，貌似并不能提高I/O读写速度呀？
数据拷贝是非常简单的，利用通用的CPU来进行数据拷贝，反倒没有使用"专项专用"的DMA高效，毕竟针对数据拷贝，DMA可以做大量的优化，让拷贝性能达到极致。目前绝大部分的计算机都默认支持DMA技术。调用read()、write()函数进行普通的I/O读写，底层已经在使用DMA技术了。

## mmap和零拷贝是如何提高I/O读写速度的？
不管读还是写，即便存在DMA，都需要进行2次数据的拷贝和1次系统调用，而1次系统调用又会导致2次用户态和内核态的上下文切换。利用mmap和零拷贝技术可以对上述读写过程进行优化，以此来提高I/O读写速度。

### mmap
mmap（memory-mapped file，内存映射文件）是提高文件读写性能的有效技术。注意，mmap一般用于文件，像网络这种数据未知的I/O设备，不适合使用mmap。

mmap的实现原理，跟普通I/O读写的底层实现原理，已经完全不同了，不再强调区分内核缓冲区和应用程序缓冲区这两个概念了，取而代之，我们需要对操作系统中的物理内存、虚拟内存、缺页中断有一定了解。这里我们简单介绍一下这三个概念。

物理内存被操作系统中同时运行的多个进程所共享，你占几块，我占几块，他占几块...每个进程都要记录自己占用了哪几块。进程操作这些不连续的内存地址会比较复杂，于是，操作系统便在物理内存之上，抽象出来了虚拟内存的概念。每个进程都有一个独享的虚拟内存，地址从0开始并且是连续的，操作系统负责记录虚拟内存跟物理内存之间的映射关系。这样每个进程只需要操作虚拟内存地址即可，操作系统负责将虚拟内存地址转化成物理地址。

除此之外，操作系统在执行程序时，并不是把所有数据都加载到物理内存再执行。毕竟物理内存是有限的，并且操作系统还要同时运行多个程序。操作系统只会在物理内存中加载程序需要的一小段数据。如果CPU在执行代码的过程中，发现需要的数据没有在物理内存中，那么，就会向操作系统发出一个缺页中断。操作系统接收到缺页中断之后，会将需要的数据从磁盘加载到物理内存。如果物理内存中没有空闲空间存储即将加载的数据，操作系统会将不再被程序需要的数据置换出物理内存，并回写到磁盘。利用这种置换（swap）机制，即便物理内存很小，我们也可以运行很大的程序。

mmap的底层实现原理
虚拟内存空间分为内核空间和用户空间。mmap将文件或者文件中的某段（由应用程序来定，待会有示例），映射到用户空间中的某段虚拟内存地址上。当应用程序读写这段虚拟内存地址对应的虚拟内存空间时，如果对应的文件没有加载到物理内存，就会触发缺页中断，然后操作系统自动将其加载到物理内存中。如果写操作改变了物理内存，在一定时间之后，操作系统会自动将“脏页”回写到磁盘。当然，我们也可以调用msync()系统调用，主动要求操作系统在写操作完成之后，立即将“脏页”回写到磁盘。

为了更好的解释mmap的实现原理，我们编写一个Linux系统下mmap的示例代码，如下所示，mmap()函数返回的ptr指针，存储的就是文件映射到虚拟内存之后的首地址。使用mmap技术，我们不需要使用read()、write()函数，直接操作虚拟内存空间（也就是操作ptr），即可实现对文件的读写。
~~~c
int main(void){
    char* file = "/users/root/in.txt";
    int fd = open(file, O_RDWR, 0666);
    if(fd < 0){
        printf("open file failed!\n");
        return -1;
    }

    // 映射文件开头（offset=0）的512字节（length=512）到ptr
    size_t length = 512;
    int offset = 0;
    char *ptr = mmap(null, length, PROT_READ | PROT_WRITE, MAP_SHARED, fd, offset);
    if (ptr == MAP_FAILED) {
        printf("mmap failed.");
        return -1;
    }
    // 创建好内存映射文件之后，fd就没用了，可以释放了
    close(fd);

    // 操作ptr就等同于读写文件
    for (int i = 0; i < length; i++) {
       ptr[i] = 'a' + (length%26);
    }
    for (int i = 0; i < N; i++) {
        printf("%c",ptr[i]);
    }
    
    // 删除内存映射文件，释放占用的虚拟内存空间
    munmap(ptr, length);
    return 0;
}
~~~

上述示例代码是在Linux操作系统下用C语言实现的，实际上，Java NIO类库也提供了相应的函数来实现mmap，具体的函数定义如下所示。实际上，在Linux操作系统下，Java中的map()函数底层也是通过调用了Linux操作系统的mmap()系统调用来实现的。
~~~java
// 位于java.nio.FileChannel类中
public MappedByteBuffer map(MapMode mode, long position, long size);
~~~

mmap相当于直接将数据在磁盘和用户空间之间互相拷贝，相对于使用read()、write()系统调用读写文件，数据拷贝次数由2次减少为1次。除此之外，使用mmap读写文件，只需要在开始时，调用一次mmap()系统调用建立好映射关系，之后读写文件就像读写内存一样，并不需要使用read()、write()系统调用，这也减少系统调用引起的用户态和内核态上下文切换的耗时。

那么，相比于使用read()、write()读写文件，使用mmap读写文件是不是就一定性能更高呢？显然，这个答案是否定的。不然，为什么还会有那么多项目使用read()、write()来读写文件呢？

我们拿插入排序和快速排序做类比。快速排序的时间复杂度要比插入排序的时间复杂度低，对于大数据量的排序，快速排序便能发挥优势，运行的更快。但是，快速排序的逻辑更加复杂，对于小数据量的排序，插入排序因为逻辑简单反而运行的更快。这里的mmap就相当于快速排序，普通的read()、write()就相当于插入排序。mmap能够减少一次数据拷贝，但这并不是免费的。mmap实现原理更加复杂。使用mmap将会带来一些额外的开销，比如建立虚拟内存与文件之间的映射会比较耗时，缺页中断导致上下文切换也会比较耗时。对于少量文件读写，使用read()、write()函数更加合适。尽管使用read()、write()函数需要进行两次数据拷贝，但是拷贝的数据量都很小，并不会太影响读写性能。对于大文件的读写，数据拷贝非常影响读写性能，因此，使用mmap的优势就更加明显。一般来讲，在项目中使用mmap之前，我们需要做一些测试来验证，其是否能真正提高读写性能。

除此之外，mmap还可以用于进程间通信。当两个应用程序都采用MAP_SHARED模式创建匿名的内存映射文件时，如下示例代码所示，这两个应用程序就会共享同一块物理内存。一个应用程序可以读取另一个程序写入物理内存的数据，以此来实现互相通信。
~~~c
char* ptr = mmap(NULL, length, PROT_READ | PROT_WRITE,
                  MAP_SHARED | MAP_ANONYMOUS, -1, 0); //fd=-1
~~~

### 零拷贝
mmap和零拷贝都可提高I/O的读写速度，只不过，它们作用的场景不同。mmap主要用于文件读写，而零拷贝（Zero-copy）主要用于两个I/O设备之间互相传输数据。特别是在将文件中的数据发送到网络或者将从网络接收的数据存储到文件这一场景中经常会用到零拷贝技术。

按照上一节讲到的普通的I/O读写流程，如果我们将文件中的一块数据读出，并通过网络发送出去，如下代码所示，需要先调用read()系统调用，再调用write()系统调用，那么总共需要执行4次数据拷贝、4次用户态和内核态之间的上下文切换（由2次系统调用引起的）。零拷贝技术的目的就是减少这个过程中的数据拷贝和上下文切换。
~~~c
read(fd, buffer,len);
write(socketfd, buffer,len);
~~~

对比普通I/O的工作流程，零拷贝并不是完全没有拷贝，而是减少了不必要的拷贝次数。使用零拷贝技术，我们不需要将数据从内核读缓冲区拷贝到应用程序缓冲区，而是直接从内核读缓冲区拷贝到内核写缓冲区。这样就节省了1次数据拷贝。除此之外，为了实现将文件发送到网络，普通的I/O流程需要调用2次系统调用（先执行read()，再执行write()），而使用零拷贝技术，应用程序只需要调用一次系统调用（执行sendfile()）。系统调用减少了一次，用户态和内核态的上下文切换减少了2次，性能也就得到了提高。

Linux操作系统提供了sendfile()系统调来实现零拷贝技术。除了文件到网络的数据传输，对于两个文件之间的数据传输，我们也可以使用零拷贝技术。示例代码如下所示。
~~~c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/sendfile.h>
#include <sys/stat.h>
#include <sys/types.h>
int main (int argc, char* argv[]) {
  int read_fd;
  int write_fd;
  struct stat stat_buf;
  off_t offset = 0;
  read_fd = open (argv[1], O_RDONLY); 
  fstat (read_fd, &stat_buf);
  write_fd = open (argv[2], O_WRONLY | O_CREAT, stat_buf.st_mode);     
  sendfile (write_fd, read_fd, &offset, stat_buf.st_size);
  close (read_fd);
  close (write_fd);
  return 0;
}
~~~

不同的操作系统提供了不同的系统调用来实现零拷贝。Java作为跨平台语言，封装了不同操作系统的系统调用的区别，提供了统一函数来实现零拷贝。在Java中，我们使用FileChannel类的transferTo()函数和transferFrom()函数来实现零拷贝。两个函数的定义如下所示。
~~~java
// 位于java.nio.FileChannel类中
public abstract long transferTo (long position, long count, WritableByteChannel target);

public abstract long transferFrom (ReadableByteChannel src, long position, long count);
~~~

实际上，内核读缓冲区到内核写缓冲区的数据拷贝也是没有必要的。有一种特殊的DMAC协处理器：SG-DMAC（Scatter-Gatter DMA Controller）可以解决这个问题。不过这需要特殊硬件的支持。在使用SG-DAMC之后，零拷贝的工作流程变成通过SG-DMAC拷贝IO设备的数据到内核缓冲区并且再拷贝内核缓冲区到IO设备，这样没有区分内核读写缓冲区，又减少了1次内核读缓冲区拷贝到内核写缓冲区的拷贝






