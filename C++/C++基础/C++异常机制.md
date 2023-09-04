 
### 栈解旋：异常被抛出后，从进入try块起，到异常被抛出前，这期间在栈上构造的对象，都会被自动析构。析构的顺序与构造的顺序相反，这一过程称为栈的解旋
 

## 异常的基本使用与自定义异常

### 注意任何类都可以作为异常类，不一定需要继承标准异常库的excetpion类
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <stdexcept> // 使用标准异常引入
#include <string> 
using namespace std;

// 注意任何类都可以作为异常类，不一定需要继承标准异常库的excetpion类
class MyException :public exception {
public:
	string message;

	MyException(string message) {
		this->message = message;
	}

	// 重写what函数
	_NODISCARD virtual char const* what() const
	{
		return this->message.c_str();
	}

};

int fun1(int a) {
	// throw关键字抛出异常，异常可以是任意类型
	// 异常会被外层的catch捕获处理，如果捕获不到则沿着方法栈向上抛，直到有捕获到异常并且处理异常
	// 异常没有捕获处理或者是捕获了不处理就会调用 terminate 函数，让程序中断
	switch (a) {
	case 1:
		throw a;
	case 2:
		throw 'a';
	case 3:
		// 注意捕获到new创建的异常对象指针处理完异常后需要使用delete回收内存空间
		throw new MyException("自定义的异常类异常信息3");
	case 4:
		// 注意栈上创建的对象，它们的指针不能作为异常对象抛出去，因为这是野指针
		// 注意如果抛出的异常对象在栈上，使用对象类型本身来捕获异常对象会发送一次拷贝，拷贝会浪费一些时间，
		// 并且异常对象和拷贝的异常对象都会在捕获异常处理异常后自动调用析构函数。
		// 当然如果抛出的异常对象在栈上，使用对象类型的引用来捕获异常对象不会发送拷贝，
		// 但是异常对象仍然会在捕获异常对象的引用后处理异常后自动调用异常对象的析构函数
		throw MyException("自定义的异常类异常信息4");
	default:
		throw "意外的异常";
	}
}

// 在函数后写  throw (异常类型1, 异常类型2)  
// 限制函数可以抛出的异常类型，如果函数体中抛出其他类型的异常会中断程序
int fun2(int a) throw (int, string) {
	switch (a) {
	case 1:
		throw a;
	default:
		throw "函数声明可能抛出指定类型的异常";
	}
}


int main() {

	try {         // try包裹可能抛出异常的代码块
		fun1(4);
	}
	catch (int) {  // catch捕获指定类型的异常进行处理
		cout << "捕获到了int类型异常" << endl;
	}
	catch (char) {
		cout << "捕获到了char类型异常，但是不处理会调用 terminate 函数中断程序" << endl;
		// 在catch中使用 throw; 就代表不处理异常继续抛出异常，而不使用 throw; 就代表被捕获的异常处理了
		throw;
	}
	catch (MyException* e) {
		cout << "捕获到了自定义异常类：" << e->what() << endl;
		// 注意new的异常对象需要手动释放
		delete e;
		e = NULL;
	}
	catch (MyException& e) {
		// 捕获异常对象的引用
		cout << "捕获到了自定义异常类：" << e.what() << endl;
	}
	catch (...) {
		cout << "捕获到了其他类型的异常" << endl;
	}

	try {
		fun2(1);
	}
	catch (string& s) {
		cout << s << endl;
	}
	catch (int) {
		cout << "捕获int类型的异常" << endl;
	}

	return 0;
}
~~~
---
 
### 系统异常的使用
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream>  // 标准输入输出
#include <stdexcept> // 使用标准异常引入
#include <string> 
using namespace std;

class Person {
public:
	int age;

	Person(int age) {

		if (age > 0 && age < 100) {
			this->age = age;
		}else {
			// 抛出标准异常库中的异常对象
			throw out_of_range("age必须在0到100之间");
		}
	}
};

int main() {

	try {
		Person(101);
	}catch (out_of_range &e) {
		cout << "所有异常类的祖先类都是exception ，而exception类的what函数获取异常的message" << endl;
		cout << e.what() << endl;
	}catch (logic_error &e) {
		cout << e.what() << endl;
	}catch (exception &e) {
		cout << e.what() << endl;
	}

	return 0;
}
~~~
---
 