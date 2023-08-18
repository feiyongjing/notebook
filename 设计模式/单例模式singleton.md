# Singleton(单例)
意图：保证一个类只有一个实例，并提供一个访问一个它的全局访问点  
常见的单例模式：Hibernate/MyBatis的sessionFactory，Spring的Bean，Java的Runtime等

## 单例模式的优点  

- 提供了对唯一实例的受控访问  
- 对于频繁访问的对象，可以省略创建对象所花费的时间和所占据的空间，是一笔非常可观的系统开销  
- 对于Java语言来说由于new次数的减少，因而对系统内存的占有减少，减轻了GC的压力，缩短了GC停顿时间  
---
# 单例模式的7种常见方式

1. ### 饿汉式：一开始类加载的时候创建单例对象  
- 静态常量  
- 静态方法  
- 静态代码块  
2. ### 懒汉式：只有在真正使用的使用的时候才会创建单例对象  
- 线程不安全  
- 线程安全 同步线程不安全的方法，缺点是效率低  
- 双重检查 Double Check （使用volatile修饰instance属性，保证多线程的可见性，即读取的永远是最新的）  
3. ### 静态内部类(推荐使用)  
4. ### 枚举  
---
## 饿汉式实现
~~~java
public class Singleton{
    private Singleton(){} //构造器私有化，保证在外部无法创建对象，然后只能在内部创建对象
    private static final Singleton INSTANCE=new Sinleton();  //静态常量方式实现：改为public访问单例对象
    public static Singleton getInstance(){ return INSTANCE; } //静态方法方式实现：对静态常量方式实现的包装
    static｛
        INSTANCE=new Singleton();
       ｝// 静态代码块的实现：声明private static final Singleton INSTANCE 不进行创建对象，在静态代码块中对 私有属性 INSTANCE 进行初始化创建对象
}
~~~

## 懒汉式实现
~~~java
public class Singleton{
    private Singleton(){} //构造器私有化，保证在外部无法创建对象，然后只能在内部创建对象
    private static Singleton instance;  //单例对象声明，还未创建出来
    public static Singleton getInstance(){ 
        if(instance==null){
            instance=new Singleton();  //多个线程同时到这一行会创建多个线程
        }
        return instance; 
    } //线程不安全的实现
    public static synchronize Singleton getInstance(){ 
        if(instance==null){
            instance=new Singleton();  
        }
        return instance; 
    } //线程安全的实现:对方法进行了加锁，但是只进行了一次“写”操作，“读”操作效率低
    public static Singleton getInstance(){ 
        if(instance==null){
            synchronize(Singleton.class){
                if(instance==null){
                    instance = new Singleton();  
                }
            }
        }
        return instance; 
    } //双重检查的实现:第一重对单例对象进行检查，保证“读”操作的效率，加锁二重检查保证“写”操作不会多次写入
    //双重检查的缺点：单例对象创建好了之后可能出现单例对象的属性为null,是指定重排序的问题
    //双重检查优化：使用volatile关键字修饰 单例对象instance ,避免重排序问题
}
~~~

## 静态内部类实现
~~~java
public class Singleton{
    private Singleton(){} //构造器私有化，保证在外部无法创建对象，然后只能在内部创建对象
    private static class SingletonInstance{
        private static final Singleton INSTANCE=new Sinleton(); 
    } 
    public static Singleton getInstance(){ return SingletonInstance.INSTANCE; } //静态内部类实现
}
~~~

## 枚举
~~~java
public enum Singleton{
    INSTANCE;//单元素的枚举单例对象
}
~~~
 