# bulider(构建)

用于创建复杂的对象，主要是类构造器多、成员繁琐  
推荐使用lombok插件简化代码

## 自己实现例子（繁琐，lombok实现例子在后面）
~~~java
public class Cat{
    String name;
    Integer age;
    Integer height;
    Integer weight;

    @Override
    public String toString() {
        return "Cat{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", height=" + height +
                ", weight=" + weight +
                '}';
    }

    public Cat(CatBulider catBulider) {
        this.name = catBulider.name;
        this.age=catBulider.age;
        this.height=catBulider.height;
        this.weight=catBulider.weight;
    }

    static class CatBulider{
    String name;
    Integer age;
    Integer height;
    Integer weight;

    public CatBulider setName(String name) {
        this.name = name;
        return this;
    }

    public CatBulider setAge(Integer age) {
        this.age = age;
        return this;
    }

    public CatBulider setHeight(Integer height) {
        this.height = height;
        return this;
    }

    public CatBulider setWeight(Integer weight) {
        this.weight = weight;
        return this;
    }
}
    public static void main(String[] args) {
        Cat cat=new Cat(new CatBulider()
                        .setName("")
                        .setAge(1)
                        .setHeight(20)
                        .setWeight(400));
        System.out.println(cat.toString());
    }
}
~~~

## lombok插件实现例子
~~~java
@Data  //省略了get set toString 构造器的代码
@Builder //标注的类使用构建器模式
public class Cat{
    String name;
    Integer age;
    Integer height;
    Integer weight;

    public static void main(String[] args) {
        Cat cat=Cat.builder()
                .name("小白")
                .age(1)
                .height(20)
                .weight(400)
                .build();
        System.out.println(cat.toString());
    }
}
~~~
 