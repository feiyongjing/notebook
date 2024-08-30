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


