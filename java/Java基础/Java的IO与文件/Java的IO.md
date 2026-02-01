## java 中 IO 流分为几种？
- 按功能来分：输入流（InputStream）、输出流（OutputStream）。
- 按类型来分：字节流和字符流。
- 字节流和字符流的区别是：字节流按8位传输以字节为单位输入输出数据，字符流按16位传输以字符为单位输入输出数据。

## File类
- File对象并不代表⼀个“⽂件”，它只代表⼀个“路径”
- 抽象的“⽂件”路径：⽂件或者⽂件夹 

## FileInputStream和FileOutputStream是文件的输入和输出

# Files是NIO的输入和输出

## File类构造器
~~~java
// 通过给定的父抽象路径名和子路径名字符串创建一个新的File实例。
File(File parent, String child);

// 通过将给定路径名字符串转换成抽象路径名来创建一个新 File 实例。
File(String pathname)

// 根据 parent 路径名字符串和 child 路径名字符串创建一个新File 实例。
File(String parent, String child) 

// 通过将给定的 file: URI 转换成一个抽象路径名来创建一个新的 File 实例。
File(URI uri)
~~~
---

## 文件的读取和写入流
- 使用FileInputStream和FileOutputStream对文件读
- 它们继承于InputStream和OuyputStream

## FileInputStream构造器
~~~java
// 将文件类代表的文件化做输入流
FileInputStream(File file)

// 根据文件路径将文件化做输入流
FileInputStream(String name)
~~~
---

## FileOutputStream构造器
~~~java
// 将文件类代表的文件化做输出流
FileOutputStream(File file)

// 如果append为 true，则将字节写入文件的末尾而不是开头
FileOutputStream(File file, boolean append)

// 根据文件路径将文件化做输出流
FileOutputStream(String name)

// 如果append为 true，则将字节写入文件的末尾而不是开头
FileOutputStream(String name, boolean append)
~~~
---

**BIO、NIO、AIO 有什么区别？**
- BIO：Block IO 同步阻塞式 IO，就是我们平常使用的传统 IO，它的特点是模式简单使用方便，并发处理能力低。
- NIO：New IO 同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，实现了多路复用。
- AIO：Asynchronous IO 是 NIO 的升级，也叫 NIO2，实现了异步非堵塞 IO ，异步 IO 的操作基于事件和回调机制。

