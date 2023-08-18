# prototype(原型)
- 指对一个对象进行深拷贝然后改变一些原有属性
- 在Java中的拷贝功能是Cloneable接口
- 任何实现了Cloneable接口，重写clone方法的类都具有克隆功能
- clone方法定义了克隆的具体实现

## 浅拷贝
clone方法里直接返回super.clone();
## 深拷贝
clone方法中使用序列化来实现深拷贝（推荐使用，并尽量使用第三方工具）
clone方法中先进行浅拷贝然后对内部引用成员属性进行浅拷贝如果内部又出现了引用成员属性继续进行浅拷贝，直到没有引用类型成员

## Java对象序列化
将实现Serializable接口的对象转换成一个字节序列，并能将字节序列反序列化成原来的对象
序列化的对象的内部属性如果是引用数据类型也需要实现Serializable接口

# 序列化/反序列化步骤
## 序列化
- 创建OutputStream对象
- 封装到ObjectOutputStream对象内
- 调用writeObject()方法
## 反序列化
- 将一个InputStream封装到ObjectInputStream对象内
- 调用readObject()方法
- 注意在类的成员属性前加关键字transient修饰时则进行序列化时对象中被修饰的成员属性所对应的值是null

例子
~~~java
class Cat implements Serializable{
    String name;

    @Override
    public String toString() {
        return "Cat{" +
                "name='" + name + '\'' +
                '}';
    }

    public Cat(String name) {
        this.name = name;
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Cat cat=new Cat("小白"); //原有对象
        
        ByteArrayOutputStream byteArrayOutputStream =new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream=new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(cat); //序列化对象写入objectOutputStream

        ByteArrayInputStream inputStream=new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        ObjectInputStream objectInputStream=new ObjectInputStream(inputStream);
        Cat cat1=(Cat) objectInputStream.readObject();//反序列化读取对象

        System.out.println(cat);
        System.out.println(cat1);
    }
}
~~~

## 序列化工具
- Jackson
- Gson
- Fast json  

参考：https://www.cnblogs.com/Vincent-yuan/p/14584581.html