# this与super
this关键字是由于对象内部的作为指针指向调用函数的对象
 

## this的使用
~~~C++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int a;
	mutable int b;

	// 对象内部的this指针指向调用函数的对象
	Person(int a,int b) {
		this->a = a;
		this->b = b;
	}

	void fun2() {
		// 为了防止使用指向NULL的对象调用函数出错，所以有this指针判空
		if (this == NULL) {
			return;
		}
		cout<< this->a <<endl;
	}
};

int main() {

	Person* p = new Person(10,20);
	cout << p->a << endl;
	cout << p->b << endl;
	
	// 指向NULL的指针调用函数
	p = NULL;
	p->fun2();
	
	return 0;
}
~~~
---
 