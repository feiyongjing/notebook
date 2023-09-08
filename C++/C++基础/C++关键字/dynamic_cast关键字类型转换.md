
### static_cast关键字是强制类型转换，但并不能保证转换的安全性，因此在使用时需要谨慎。如果转换不合法，可能会导致程序崩溃或者产生不可预知的结果
### dynamic_cast关键字只能将父类指针或引用转换为子类类指针或引用，同时还会检查转换的合法性
### const_cast关键字只能用于将 const 或 volatile 类型的指针或引用转换为非 const 或非 volatile 类型的指针或引用
### reinterpret_cast关键字是强制类型转换，用于将一个指针或引用转换为另一种指针或引用。
 
~~~C++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

class Animal {
public:
	string name;
	
	Animal(string name) {
		this->name = name;
	}

	void fun1() {
		cout << "父类函数调用" << endl;
	}
};

class Person :public Animal{
public:
	int age;

	Person(string name,int age):Animal(name) {
		this->age = age;
	}

	void fun1() {
		cout << "子类函数调用" << endl;
	}
};

int main() {
	Animal a("张三父亲");
	Person p("张三", 18);

	cout << a.name << endl;
	cout << p.name << endl;

	// 父类类型的引用和指针可以指向子类类型的对象
	Animal& a1 = p;
	Animal* a2 = &p;

	// static_cast关键字是强制类型转换，但并不能保证转换的安全性，因此在使用时需要谨慎。如果转换不合法，可能会导致程序崩溃或者产生不可预知的结果
	// static_cast关键字可以对基本数据类型转换
	// 语法： static_cast<想要转换的类型>(变量);
	Person* p1 = static_cast<Person*>(&a);
	cout << "例如父类对象使用static_cast关键字强制转换为子类对象会出现未知的现象：" << p1->age << endl;

	// dynamic_cast关键字只能将父类指针或引用转换为子类类指针或引用，同时还会检查转换的合法性
	// 主要用于多态类型的转换，即将父类指针或引用转换为子类指针或引用。
	// 在进行这种类型转换时，需要保证父类指针或引用所指向的对象实际上是子类的对象，否则不合法转换会失败
	// 语法： dynamic_cast<想要转换的类型>(变量);
	Animal* a3 = dynamic_cast<Animal*>(&p);
	cout << a3->name << endl;

	// const_cast关键字只能用于将 const 或 volatile 类型的指针或引用转换为非 const 或非 volatile 类型的指针或引用
	// 需要注意的是，const_cast 仅能用于去除指针或引用的常量性或易变性，不能用于去除变量本身的常量性或易变性。
	// 即常量指针和引用可以变为非常量的指针和引用，但是常量内部的数据还是不能修改
	// 例如如下代表虽然编译可以通过，但是运行时常量的值不会变化
	// 语法： const_cast<想要转换的类型>(变量);
	const int y = 1;
	int& t = const_cast<int&>(y);
	t = 20;
	cout << y<< endl;
	cout << t << endl;

	// reinterpret_cast关键字是强制类型转换，用于将一个指针或引用转换为另一种指针或引用。
	// 它可以将任何指针类型转换为任何其他指针类型，但是这种转换是非常危险的，因为它可以导致转换的指针或引用无法正常使用
	// 例如如下转换int指针为string指针
	int n = 10;
	int* n1 = reinterpret_cast<int*>(&n);
	cout << "测试reinterpret_cast转换int指针为int指针" << *n1 << endl;
	string* m = reinterpret_cast<string*>(&n);
	cout << "测试reinterpret_cast转换int指针为string指针" << *m << endl;

	return 0;
}
~~~
---
 