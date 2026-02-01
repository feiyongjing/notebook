任何只包含一个抽象方法的接口都可以自动转换为函数接口
lambda表达式： s->{} s代表参数，{}代表方法体
方法引用：类名::方法名
----------------------------------------------------------------------------
java.util.function包
Predicate接口的test抽象方法接受一个对象返回一个布尔值
Consumer接口的accept抽象方法接受一个对象不返回任何的东西（即消费者）
Function接口的apply抽象方法接受一个对象将对象的类型转换为另一个类型返回
Supplier接口的get抽象方法从虚空变出一个对象（即生产者）
----------------------------------------------------------------------
BooleanSupplier接口的抽象方法从虚空变出一个布尔值
DoubleSupplier接口的抽象方法从虚空变出一个doubie
IntSupplier接口的抽象方法从虚空变出一个int
LongSupplier接口的抽象方法从虚空变出一个long
等等更多的Supplier接口
----------------------------------------------------------------------
在java.util.function包中Predicate、Function、Consumer都有与上述Supplier接口类似的接口
----------------------------------------------------------------------
Comparator 比较器
comparing() 比较(方法接受一个Function的实例对象)
thenComparing() 然后比较(方法接受一个Function的实例对象)
reversed() 反序
Comparator接口的comparing抽象方法接受一个Function的实例对象