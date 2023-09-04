 

父类的全部构造函数（包含拷贝构造函数）、operator=函数、析构函数不会被子类继承，原因是只有父类才知道如何构造父类对象和释放对象

创建对象时先调用父类的构造函数，再调用子类的构造函数

回收对象时先调用子类的析构函数，再调用父类的析构函数


# 多继承的缺点与解决方案
- 多个父类有类型名称一样的成员和函数，导致使用必须加作用域指定使用那个属性
- 多个父类有类型名称一样的成员，导致子类继承了两份成员，但是实际只使用一个导致有内存的浪费
- 解决方案：虚继承可以有效避免内存的浪费以及同名成员的作用域调用
 

~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Animal {
public:
	int name;

	Animal(int name) {
		this->name = name;
		cout << "父类动物类的构造函数被调用" << endl;
	}

	~Animal() {
		cout << "父类动物类的析构函数被调用" << endl;
	}

	void animalSound() {
		cout << "动物叫的声音" << endl;
	}
};

// 继承方式，C++支持多继承
// 子类:  继承方式 父类1 , 继承方式 父类2
// 继承方式有三种：public、property、pricate，注意三种继承方式子类都会继承父类的全部属性和函数
// public继承：父类中的public属性函数和property属性函数按照原有修饰符继承在子类中可以访问、父类中private属性函数子类会继承但是无法访问
// property继承：父类中的public属性函数和property属性函数在子类中会继承但是都是property修饰的、父类中private属性函数子类会继承但是无法访问
// private继承：父类中的public属性函数和property属性函数在子类中会继承但是都是private修饰的、父类中private属性函数子类会继承但是无法访问
class Person : public Animal {
public:
	// 选择调用父类的构造函数，写法如下
	// : 父类名称(父类构造参数列表)
	// 父类的构造参数列表会从子类的构造参数列表中获取
	Person(int name) : Animal(name) {
		cout << "子类人类的构造函数被调用" << endl;
	}

	~Person() {
		cout << "子类人的析构函数被调用" << endl;
	}

	void animalSound() {
		cout << "人叫的声音：&@#)！~*%^" << endl;
	}

};

class Cat : public Animal {
public:

	Cat(int name) : Animal(name) {
		cout << "子类猫的构造函数被调用" << endl;
	}

	~Cat() {
		cout << "子类猫的析构函数被调用" << endl;
	}

	void animalSound() {
		cout << "猫叫的声音：喵喵" << endl;
	}
};

int main() {
	Person p(9);
	Cat c(1);

	cout << "人的名称序号是：" << p.name << endl;
	cout << "猫的名称序号是：" << c.name << endl;

    // 子类和父类中有同名的成员或者是同名的函数时
	// 默认是调用子类的成员和函数，
	p.animalSound();
	c.animalSound();

    // 注意子类会覆盖隐藏父类中同名的全部函数，即使是重载的参数列表不匹配，只要是同名的函数就会覆盖隐藏
	// 如果需要调用父类的隐藏的成员或者是隐藏的函数请使用作用域，对象.父类::成员或函数
	p.Animal::animalSound();
	return 0;
}
~~~
---
 

# 虚继承

### 虚继承可以有效避免内存的浪费以及同名成员的作用域调用
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Animal {
public:
	int age;
};

// 羊
// virtual关键字申明虚继承
// 当发生虚继承后sheep和tuo类中继承了一个 vbptr指针（虚基类指针）指向的是一个虚基类 vbtable
// vbtable中记录了偏移量，通过偏移量可以找到唯一的age属性(当然也可以是父类其他类型名称相同的属性)
class Sheep : virtual public Animal {};

// 骆驼
// virtual关键字申明虚继承
class Tuo : virtual public Animal {};

// 羊驼
class SheepTuo : public Sheep, public Tuo {
};


int main() {

	// 使用虚继承避免内存的浪费
	SheepTuo s;
	s.age = 0;
	cout << s.age << endl;

	s.Sheep::age = 10;
	cout << s.age << endl;

	s.Tuo::age = 20;
	cout << s.age << endl;

	s.age = 30;
	cout << s.age << endl;
	return 0;
}
~~~
---
 