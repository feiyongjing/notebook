### 缓冲区是用于CPU一次读和写多个字节或字符流

# BufferedReader
~~~java
// 创建默认大小的读缓冲区，大约8000字符
BufferedReader(Reader in)
// 创建指定大小的读缓冲区
BufferedReader(Reader in, int sz)

// 从缓冲区中读取单个字符
int read()

// 从缓冲区中读取若干个字节，把它们保存到参数cbuf指定的数组中，off指定字节数组开始保存数据的起始下标，len表示需要读取的字节数目，如果由于已经到达流的末尾而没有字节可用，则返回-1 
int read(char cbuf[], int off, int len)

// 从缓冲区中读取一行文本。一行被认为由换行符（'\n'），回车符（'\r'）或回车符*中的任意一个终止，后接换行符。等同于readLine(false)
String readLine()
~~~


# BufferedWriter
~~~java
// 创建默认大小的写缓冲区，大约8000字符
BufferedWriter(Writer out)

// 创建指定大小的写缓冲区
BufferedWriter(Writer out, int sz)

// 从缓冲区中写一个字符
void write(int c)

// 从缓冲区中写若干个字节，把它们保存到参数cbuf指定的数组中，off指定字节数组开始保存数据的起始下标，len表示需要读取的字节数目，如果请求的长度至少等于缓冲区的长度，则此方法将刷新缓冲区并将字符直接写入基础流。因此，冗余 BufferedWriter 不会不必要地复制数据。
void write(char cbuf[], int off, int len)

// 写入字符串的一部分。 如果len参数的值为负，则不会写入任何字符。这与{@linkplain java.io.Writer＃write（java.lang.String，int，int）超类}中的方法的规范相反，后者要求抛出{@link IndexOutOfBoundsException}。 s要写入的字符串 off开始读取字符的偏移量，len要写入的字符数
void write(String s, int off, int len)

//写一个行分隔符。行分隔符字符串是由系统属性 line.separator定义的，不一定是单个换行符（'\ n'）
void newLine()

//刷新缓冲区并将字符直接写入基础流
void flush()

~~~

