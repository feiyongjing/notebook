# AnnotatedElement实现类
- Class：类定义
- Constructor：构造器定义
- Field：累的成员变量定义
- Method：类的方法定义
- Package：类的包定义

# 方法解析
~~~java
//返回该程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。
< T extends Annotation> T getAnnotation(Class< T> annotationClass)

//返回该程序元素上存在的所有注解。
Annotation[] getAnnotations()

//判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.
boolean isAnnotationPresent(Class< ? extends Annotation> annotationClass)

//返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。
Annotation[] getDeclaredAnnotations()
~~~