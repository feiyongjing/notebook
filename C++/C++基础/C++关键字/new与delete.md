## new和delete
- new关键字可以在堆上创建对象
- delete关键字可以回收堆上的对象

## 栈上创建对象和堆上创建对象

### 在栈上创建对象
~~~C++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int b;
public: 

	Person() {
		cout << "无参构造函数调用" << endl;
	}

	Person(int m_b) {
		b = m_b;
	}
};

int main() {
	Person p = Person(10);
	cout << p1->b << endl;

	return 0;
}
~~~
----
 

### new在堆上创建对象和delete释放new创建的对象
~~~C++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int b;
public: 

	Person() {
		cout << "无参构造函数调用" << endl;
	}

	Person(int m_b) {
		b = m_b;
	}
};

int main() {

	// new关键字在堆内存中创建对象
	Person* p = new Person();
	// delete关键字调用析构函数并清除通过new创建的对象在堆上占用内存空间
	// 注意delete无法释放void * 类型的指针，如果需要请进行强制类型转换
	delete p;

	// 上述new创建对象相当于执行以下代码
	/*Person* p = (Person*)malloc(sizeof(Person));
	if (p == NULL) {
		return 0;
	}
	// 调用自定义init函数初始化内部的一些成员，作用和调用构造函数一样
	p->init();*/

	// 上述delete关键字回收对象在堆的内存空间相当于执行以下代码
	/*if (p != NULL) {
		free(p);
		p = NULL;
	}*/

	Person* p1 = new Person(10);
	cout << p1->b << endl;
	
	// 在堆上创建数组
	int* pint = new int[10];
	// 如果数组中元素是对象，则该对象必须有无参构造函数，
	// 因为在有些编译器中在堆上创建数组内部的元素时可以调用有参的构造函数，但是有些不行
	// 而在堆上创建数组内部的元素时调用的无参构造函数是所有编译器都支持的，所以强制要求必须有无参构造函数
	Person* ps = new Person[10];

	// 释放堆上的数组，必须在delete后指针前加上 [] 表示释放的是数组，否则只释放数组的第一个元素而其他元素不释放
	delete [] pint;
	delete [] ps;
	
	// 在栈上开辟数组不强制使用默认的构造函数
	Person ps1[10] = {Person(11),Person(22)};
	return 0;
}
~~~
----
 
