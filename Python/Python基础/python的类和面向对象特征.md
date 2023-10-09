### 类的定义、构造方法和比较方法
~~~python
class User:
    name = None

    # 类中的方法参数必须含有self参数，self是一个关键字表示方法的调用者对象，类似与java的this
    # 在调用方法时不用传递self参数，默认该参数就是调用者本身
    # 在方法内部可以通过self获取和修改对象成员属性
    def say_hi(self):
        print(f"我的名称是：{self.name}")

    # __init__ 是类的构造函数，在构造函数中进行类的赋值，创建对象时会自动执行构造函数
    # 甚至类的成员定义可以不用写而是在构造函数中直接进行赋值就相当于进行了类成员的定义和赋值
    def __init__(self, age):
        self.age = age

    # __str__ 是类的字符串表示，默认不写的实现是返回对象的内存地址
    # 通过重写 __str__ 方法可以快速的查看对象的属性信息
    def __str__(self):
        return f"name是：{self.name}，age是{self.age}"

    # __lt__ 是类的对象的小于比较方法，一般情况下不写 __lt__ 方法是无法使用运算符<和>比较对象
    # 通过重写 __lt__ 方法后两个对象就可以通过<和>进行对象的比较了(其中>的比较是 __lt__ 方法结果自动取反)
    def __lt__(self, other):
        return self.age < other.age

    # __le__ 是类的对象的小于等于比较方法，一般情况下不写 __le__ 方法是无法使用运算符<=和>=比较对象
    # 通过重写 __le__ 方法后两个对象就可以通过<=和>=进行对象的比较了(其中>=的比较是 __lt__ 方法结果自动取反)
    def __le__(self, other):
        return self.age <= other.age

    # __eq__ 是类的对象比较是否相等，一般情况下不写 __eq__ 方法是使用运算符==比较对象其实是比较的对象内存地址
    # 通过重写 __eq__ 方法后两个对象就可以通过==进行对象的比较两个对象是否相等
    def __eq__(self, other):
        return self.age == other.age


user = User(18)
user.name = "张三"

user.say_hi()
print(user.age)

print("str内置函数打印对象信息，其内部本质是调用对象的__str__方法", str(user))
print("打印重写__str__方法的类对象信息", user.__str__())

user1 = User(24)
user2 = User(28)
print(user1 < user2)
print(user1 > user2)

user3 = User(30)
user4 = User(30)
print(user3 <= user4)
print(user3 >= user4)
print(user3 == user4)
~~~
---

# 类成员和方法的封装
### 类中的成员和方法（除了默认的基础方法之外的方法）如果是以两个下划线开头则这些成员和方法是受保护的，无法在外部通过对象调用
~~~python
class User:
    name = None

    # 类中的成员如果是以两个下划线开头则这些成员是受保护的，无法在外部通过对象调用获取(获取会报错)和修改，在外部调用修改不会报错但是修改不会生效
    __age = None

    # 类中的方法（除了默认的基础方法之外的方法）如果是以两个下划线开头则这些方法是受保护的，无法在外部通过对象调用，在外部调用运行会报错
    def __say_hi(self):
        print(f"我的名称是：{self.name}")

    def __init__(self, name,age):
        self.name=name
        self.age = age

user = User("张三", 18)

# 受保护的方法调用运行会报错
user.__say_hi()
# 受保护的成员修改不会报错但是修改不生效
user.__age=20

~~~
---


# 继承

### 单继承
~~~python
class Animal:
    name = None

    def __init__(self,name):
        self.name = name

# 猫继承动物
class Cat(Animal):
    def animalSound(self):
        print(f"{self.name}的叫声：喵喵")

cat =Cat("猫猫")
cat.animalSound()
~~~
---

### 多继承
~~~python
class Animal:
    name = None

    def __init__(self,name):
        self.name = name

class Sheep(Animal):
    def animalSound(self):
        print(f"{self.name}的叫声：咩咩")

class Tuo(Animal):
    def animalSound(self):
        print(f"{self.name}的叫声：咕咕")

# 多继承，注意多继承出现同名的成员属性或成员方法时在括号在左边的生效，右边的无效
class SheepTuo(Sheep,Tuo):
    age = None

sheepTuo =SheepTuo("羊驼")
sheepTuo.animalSound()
~~~
---

### 在子类中调用父类的成员或方法
~~~python
class Animal:
    name = "动物"

    def __init__(self,name):
        self.name = name

    def animalSound(self):
        print(f"动物叫的声音：&@#)！~*%^")

class Sheep(Animal):
    def animalSound(self):
        # 方式一：调用父类的成员或方法,注意调用方法需要传递self参数（当前对象）
        print(Animal.name)
        Animal.animalSound(self)

        # 方式二：调用父类的成员或方法
        print(super().name)
        super().animalSound()

        print(f"{self.name}的叫声：咩咩")

class Sheep(Sheep):
    age = None

sheep =Sheep("羊驼")
sheep.animalSound()
~~~
---

### 抽象类
~~~python
class Animal:
    name = "动物"

    def __init__(self,name):
        self.name = name

    # 父类函数没有具体的实现而是直接pass，含有这样抽象函数的类就是抽象类，如果类中全部都是抽象函数则是接口，子类继承抽象类和接口必须实现全部的抽象函数
    def animalSound(self):
        pass

class Sheep(Animal):
    def animalSound(self):
        print(f"{self.name}的叫声：咩咩")

class Sheep(Sheep):
    age = None

sheep =Sheep("羊驼")
sheep.animalSound()
~~~
---