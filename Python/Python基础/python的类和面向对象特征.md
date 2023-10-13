### 类的定义、构造方法和比较方法
~~~python
class User:
    name = None

    # staticmethod 是 Python 中的一个内置装饰器，它可以将一个方法转换为静态方法。
    # 静态方法是不需要访问实例或类的状态（class对象）的方法，因此它们不需要传递 self 或 cls 参数。
    # 静态方法可以通过类或实例调用，但是它们不会自动传递类或实例作为第一个参数。
    @staticmethod
    def sfn():
        print("静态方法不和class对象绑定")

    # classmethod 也是 Python 中的一个内置装饰器，它可以将一个方法转换为类方法。
    # 类方法是需要访问类状态（class对象）的方法，因此它们需要传递 cls 参数。
    # 类方法可以通过类或实例调用，但是它们会自动传递类（class对象）作为第一个参数
    @classmethod
    def cfn(cls):
        print("class方法和class对象绑定，必须有cls方法参数，cls参数就是class对象")

    # 类中的实例方法和类的对象实例绑定，参数必须含有self参数，self是一个关键字表示方法的调用者对象，类似与java的this
    # 在调用方法时不用传递self参数，默认该参数就是调用者本身
    # 在方法内部可以通过self获取和修改对象成员属性
    def say_hi(self):
        print(f"我的名称是：{self.name}")

    # __new__ 是对象创建分配内存空间的方法，会返回创建对象的引用给 __init__ 方法
    def __new__(cls, *args, **kwargs):
        return super().__new__(cls)

    # __init__ 是类的构造函数，在构造函数中进行类的赋值，创建对象时会自动执行构造函数
    # 甚至类的成员定义可以不用写而是在构造函数中直接进行赋值就相当于进行了类成员的定义和赋值
    def __init__(self, age):
        self.age = age

    # __del__ 是对象销毁前会执行的方法
    def __del__(self):
        print("对象销毁前需要做的事情")

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

# 静态方法
User.sfn()

# 类方法
User.cfn()

user = User(18)
user.name = "张三"

user.say_hi()
print(user.age)

print("str内置函数打印对象信息，其内部本质是调用对象的__str__方法", str(user))
print("打印重写__str__方法的类对象信息", user.__str__())

# del 关键字可以销毁一个对象
del user

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

# 对象动态绑定属性和方法
~~~python
class User:
    name = None
    age = None

    def __init__(self, name, age):
        self.name = name
        self.age = age


user = User("张三", 18)

# python可以在对象创建之后给对象单独绑定自定义的属性和方法
# 绑定新的属性gender
user.gender = '男'

# 绑定新的方法如果需要使用方法内部的属性必须这样写
def show(self):
    print(self.name, self.age, self.gender)

user.show = lambda :show(user)

# 绑定新的方法后就可以调用新方法了
user.show()
~~~
---

# 使用元类动态创建类
~~~python
# 定义一个简单的基类
class BaseClass:
    def greeting(self):
        print("Hello from BaseClass")

# 动态创建一个名为 "DerivedClass" 的类，继承自 BaseClass，内部有一个自定义的say_hi函数
DerivedClass = type("DerivedClass", (BaseClass,), {
    "say_hi": lambda self: print("Hi from DerivedClass")
})

# 创建 DerivedClass 的实例并调用其方法
obj = DerivedClass()
obj.greeting()  # 输出: Hello from BaseClass
obj.say_hi()    # 输出: Hi from DerivedClass
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
        print(f"我的年龄是：{self.__age}")

    def __init__(self, name,age):
        self.name=name
        self.age = age

user = User("张三", 18)

# 受保护的方法直接调用运行会报错
user.__say_hi()
# 受保护的成员直接修改不会报错但是修改不生效
user.__age=20

# 但是以上的封装只是伪封装（对封装的属性和方法做了一些处理，即在封装的属性名和方法名前面添加_类名），依然可以通过 _类名__封装的属性或方法 调用封装属性或封装方法
user._User__age=30
user._User__say_hi()

~~~
---


# 继承

### 单继承
~~~python
class Animal:
    name = None

    def __init__(self, name):
        self.name = name

# 猫继承动物
class Cat(Animal):

    age = None

    # 子类默认继承父类的构造器，旧的属性可以直接调用父类构造器赋值，新的属性需要在子类构造器中自行赋值
    def __init__(self, name, age):
        super().__init__(name)
        self.age = age

    def animalSound(self):
        print(f"{self.name}的叫声：喵喵，年龄是{self.age}")

cat =Cat("猫猫", 18)
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

# 多继承
# 注意子类对象调用多个父类中同名属性或方法时是按照__mro__属性返回的元组顺序查找类，最终会执行第一个找到类中属性或方法
# 一般是继承时在括号左边类的生效，右边的无效
class SheepTuo(Sheep,Tuo):
    age = None

sheepTuo =SheepTuo("羊驼")
sheepTuo.animalSound()
~~~
---

### python3中所有的类都默认继承于object类, python2中没有声明继承object就不会继承object类
~~~python
obj = object()
print(type(obj))

# Object类中可以进行的操作，这里面有很多特殊的属性和方法
print(dir(obj))



# 一些常见的特殊属性和方法如下
class User:
    name = None
    age = None

    def say_hi(self):
        print(f"我的名称是：{self.name}")

    # 创建对象是先执行的 __new__() 函数创建对象再执行的 __init__() 函数初始化对象
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # 自定义对象的相加
    # 实现了 __add__() 函数则对象就可以使用运算符 + 进行加了，结果就是 __add__() 函数的返回值
    def __add__(self, other):
        return self.age + other.age

    # 自定义对象的长度
    # 实现了 __len__() 函数则对象就可以使用内置函数len()计算对象的长度，结果就是 __len__() 函数的返回值
    def __len__(self):
        return len(self.name)


user = User("张三", 18)

# __dict__属性获取对象的属性名称和值组成的字典，当然可以可以获取class对象的属性字典里面包含函数名称和函数对象
print(user.__dict__)

# __class__属性获取对象的class对象，当然也可以直接使用 type(对象) 获取 或者 直接通过类名获取
print(user.__class__)
print(type(user))
print(User)

# class对象的__bases__属性返回一个元组存放类所继承的全部父类类型
print(User.__bases__)

# class对象的__base__属性获取类所继承的主要父类，即声明继承时最左边的父类
print(User.__base__)

# class对象的__mro__属性返回一个元组存放类的继承链，内部包含类自己和全部的父类直到object类，子类对象调用多个父类中同名的方法时是按照__mro__属性返回的元组顺序查找方法所在的类找，最终会执行第一个找到类中方法
print(User.__mro__)

# class对象的__subclasses__方法返回一个元组存放当前类被那些类继承了
print(dict.__subclasses__())

# 实现了 __add__() 函数则对象就可以使用运算符 + 进行加了，结果就是__add__() 函数的返回值
user1 = User("xxx", 12)
user2 = User("eee", 19)
print(user1 + user2)

# 实现了 __len__() 函数则对象就可以使用内置函数len()计算对象的长度，结果就是 __len__() 函数的返回值
user3 = User("xxaqsx12wx", 12)
print(len(user3))
print(user3.__len__())

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