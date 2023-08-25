# InputStream常用方法
~~~java
// 从输入流中读取一个8位的字节并转换为0~255之间的整数返回，如果由于已经到达流的末尾而没有字节可用，则返回-1 
int read()	

// 从输入流中读取若干个字节，把它们保存到参数b指定的数组中，返回的整数表示读取字节数，如果由于已经到达流的末尾而没有字节可用，则返回-1 
int read(byte[] b)	

// 从输入流中读取若干个字节，把它们保存到参数b指定的数组中，off指定字节数组开始保存数据的起始下标，len表示需要读取的字节数目，如果由于已经到达流的末尾而没有字节可用，则返回-1 
int read(byte[] b, int off, int len)	

// 关闭此输入流并释放与该流关联的所有系统资源
void close()	

~~~


# OutputStream常用方法
~~~java

// 向输入流写入一个字节
void write(int b)	

// 把参数b指定的字节数组的所有字节写入到输出流
void write(byte[] b)	

// 将指定的byte数组中从偏移量off开始的len个字节写入输出流
void write(byte[] b, int off, int len)	

// 刷新此输出流并强制写出所有缓冲的输出字节
void flush()	

// 关闭此输出流并释放与该流关联的所有系统资源
void close()	

~~~