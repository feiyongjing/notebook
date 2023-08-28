

### try/catch/finally
- 如果没有try/catch，异常将击穿所有的栈帧
- catch可以将⼀个异常抓住
- finally执⾏清理⼯作
### 注意事项
- catch 不能独立于 try 存在。
- 在 try/catch 后面添加 finally 块并非强制性要求的。
- try 代码后不能既没 catch 块也没 finally 块。
- finally 块不要出现方法语句
- try, catch, finally 块之间不能添加任何代码
### JDK7+：try-with-resources
1、当一个外部资源的句柄对象实现了AutoCloseable接口，JDK7中便可以利用try-with-resource语法更优雅的关闭资源，消除板式代码。
2、try-with-resource时，如果对外部资源的处理和对外部资源的关闭均遭遇了异常，“关闭异常”将被抑制，“处理异常”将被抛出，但“关闭异常”并没有丢失，而是存放在“处理异常”的被抑制的异常列表中。
3、 例子如下
~~~java
    public static void main(String[] args) {
        try (FileInputStream inputStream = new FileInputStream(new File("test"))) {
            System.out.println(inputStream.read());
        } catch (IOException e) {
            throw new RuntimeException(e.getMessage(), e);
    }
~~~


### 声明自定义异常，在 Java 中你可以自定义异常。编写自己的异常类时需要记住下面的几点。
- 所有异常都必须是 Throwable 的子类。
- 如果希望写一个检查性异常类，则需要继承 Exception 类。
- 如果你想写一个运行时异常类，那么需要继承 RuntimeException 类。


### Java的异常体系
- Throwable 指可以被抛出的东⻄
  - Error （错误，指不受检查的异常，并且是无法恢复的异常）
  - Exception 指checked execption（受检异常）
    - RuntimeException （运⾏时异常，编译时不会进行检查）
    - IOException（编译时受检异常）

# Java 常见异常
### Java 不受检异常
- ArithmeticException	当出现异常的运算条件时，抛出此异常。例如，一个整数"除以零"时，抛出此类的一个实例。
- ArrayIndexOutOfBoundsException	用非法索引访问数组时抛出的异常。如果索引为负或大于等于数组大小，则该索引为非法索引。
- ArrayStoreException 	试图将错误类型的对象存储到一个对象数组时抛出的异常。
- ClassCastException	当试图将对象强制转换为不是实例的子类时，抛出该异常。
- IllegalArgumentException	抛出的异常表明向方法传递了一个不合法或不正确的参数。
- IllegalMonitorStateException 	抛出的异常表明某一线程已经试图等待对象的监视器，或者试图通知其他正在等待对象的监视器而本身没有指定监视器的线程。
- IllegalStateException	在非法或不适当的时间调用方法时产生的信号。换句话说，即 Java 环境或 Java 应用程序没有处于请求操作所要求的适当状态下。
- IllegalThreadStateException	线程没有处于请求操作所要求的适当状态时抛出的异常。
- IndexOutOfBoundsException	指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出
- NegativeArraySizeException 	如果应用程序试图创建大小为负的数组，则抛出该异常。
- NullPointerException	当应用程序试图在需要对象的地方使用 null 时，抛出该异常
- NumberFormatException	当应用程序试图将字符串转换成一种数值类型，但该字符串不能转换为适当格式时，抛出该异常。
- SecurityException	由安全管理器抛出的异常，指示存在安全侵犯。
- StringIndexOutOfBoundsException	 此异常由 String 方法抛出，指示索引或者为负，或者超出字符串的大小。
- UnsupportedOperationException	当不支持请求的操作时，抛出该异常。

### Java 受检异常
- ClassNotFoundException	应用程序试图加载类时，找不到相应的类，抛出该异常。
- CloneNotSupportedException	当调用 Object 类中的 clone 方法克隆对象，但该对象的类无法实现 Cloneable 接口时，抛出该异常。
- IllegalAccessException	拒绝访问一个类的时候，抛出该异常。
- InstantiationException	当试图使用 Class 类中的 newInstance 方法创建一个类的实例，而指定的类对象因为是一个接口或是一个抽象类而无法实例化时，抛出该异常。
- InterruptedException	一个线程被另一个线程中断，抛出该异常。
- NoSuchFieldException	请求的变量不存在
- NoSuchMethodException	请求的方法不存在

