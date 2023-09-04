### C++支持编译时多态和运行时多态，类中函数重载和运算符重载就是编译时多态，但是通常说的多态是指运行时多态，而子类和虚函数就是运行时多态
- 编译时多态是指在代码编译时就知道函数的调用地址
- 运行时多态是指代码在运行时才能获取到函数的调用地址

# 多态调用
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Animal {
public:

	void animalSound() {
		cout << "动物叫的声音" << endl;
	}
};

class Dog : public Animal {
public:

	void animalSound() {
		cout << "狗叫的声音：&@#)！~*%^" << endl;
	}

};

class Cat : public Animal {
public:

	void animalSound() {
		cout << "猫叫的声音：喵喵" << endl;
	}
};


void speak(Animal & a) {
	// 由于变量a是Animal类型所以调用的是父类animalSound函数，
	// 除非父类Animal的animalSound函数使用 virtual 关键字声明是虚函数，在这种情况下变量a的实际类型是子类就会调用子类的animalSound函数
	a.animalSound();
}

int main() {
	Dog d;
	Cat c;

	speak(d);
	speak(c);

	d.animalSound();
	c.animalSound();


	return 0;
}
~~~
---
 

 多态释放时调用析构函数
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Animal {
public:
	
	Animal() {
		cout << "父类动物类的构造函数被调用" << endl;
	}

	virtual ~Animal() {
		cout << "父类动物类的析构函数被调用" << endl;
	}

	void animalSound() {
		cout << "动物叫的声音" << endl;
	}
};

class Cat : public Animal {
public:
	char* name;

	Cat(const char* name){
		cout << "子类猫的构造函数被调用" << endl;
		// 将name属性放入堆内存
		char* m_name = (char*)malloc(sizeof(name) + 1);
		strcpy(m_name, name);
	}

	~Cat() {
		cout << "子类猫的析构函数被调用" << endl;
		if (name != NULL) {
			free(name);
			name = NULL;
		}
	}

	void animalSound() {
		cout << "猫叫的声音：喵喵" << endl;
	}
};

int main() {
	Animal* c=new Cat("yyy");

	c->animalSound();
	
	// 不进行强制类型转为子类类型，而使用父类类型变量释放对象，
	// 如果父类Animal没有virtual关键字修饰析构函数，那么只调用父类的析构函数，不调用子类的构造函数，可能导致子类存放在堆上的成员没有释放
	// 如果父类Animal有使用virtual关键字修饰析构函数，那么先调用子类的析构函数再调用父类的析构函数
	// 当然如果明确释放对象的具体类型就不会出现上述问题
	delete c;

	return 0;
}
~~~
---
 