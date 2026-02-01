# File常用方法
~~~java
// 返回从此抽象路径构造的Path。在File中有对应的toFile()方法
Path toPath()

// 创建以此抽象路径名命名的目录
boolean mkdir()

// 创建以此抽象路径名命名的目录，包括任何必需但不存在的父目录
boolean mkdirs()

// 判断File对象表示的文件或目录存在
boolean exists()	

// 判断File对象表示的东西是不是一个文件
boolean isFile()	

// 判断File对象表示的东西是不是一个文件夹(即目录)
boolean isDirectory()	

// 删除File对象对应的文件或目录
boolean delete()	

// 判断File对象对应的文件或目录是否可读
boolean canRead()	

// 判断File对象对应的文件或目录是否可写
boolean canWrite()	

// 判断File对象对应的文件或目录是否是绝对路径
boolean isAbsolute()	

// 当File对象对应的文件不存在时，创建文件，并将新建一个File对象所指定的新文件
boolean createNewFlie()	

// 返回File对象表示的文件或文件夹的名称
String getName()	

// 返回File对象对应的路径
String getPath()	

// 返回File对象的绝对路径名字符串。
String getAbsolutePath()	

// 返回File对象对应目录的父目录
String getParent()	

// 列出指定目录的全部内容，只列出来名称 
String[] list() 	

// 返回一个包含File对象所有子文件和子目录的File数组
File[] listFile()	

// 返回文件的长度
long length()	

// 返回1970年1月1日0时0分0秒到文件最后修改时间的毫秒值
long lastModified()	

~~~

# Files常用方法

~~~java
// 检测文件路径是否存在
Files.exists()

// 创建文件
Files.createFile()

// 创建文件夹
Files.createDirectory()

// 删除一个文件或目录
Files.delete()

// 复制文件
Files.copy()

// 移动文件
Files.move()

// 查看文件个数
Files.size()

// 从文件读取所有字节。将所有字节读入字节数组返回
byte[] readAllBytes(Path path)

// 读取文件中的所有行。将所有行收集起来返回一个List,等价于readAllLines(path, StandardCharsets.UTF_8)方法
List<String> readAllLines(Path path)

// 与readAllLines(Path path)一样，参数cs是用于解码的字符集
List<String> readAllLines(Path path, Charset cs)

// 它打开文件进行写入，参数options指定如何打开文件的选项，如果不存在则创建文件，或者首先将现有的{常规文件}截断。在创建之后将字节数组中的所有字节均写入文件。该方法可确保在写入所有字节后（或者引发I/O错误或其他运行时异常）关闭文件。如果发生I/O错误，则可以在创建或截断文件之后或在将某些字节写入文件后执行。
Path write(Path path, byte[] bytes, OpenOption... options)

~~~


