### 重载运算符
- 运算符使用时会调用特定的函数
- 分为全局函数重载运算符和类中函数重载运算符
- 使用operator关键字重载运算符
- 同一运算符允许多次重载
- 只能重载涉及自定义类的运算符，全部是基本数据类型的运算时，运算符不允许重载
- 特殊的运算符重载需要注意，例如++ 和 -- 在前面和后面时，例子在下面
- 特殊的运算符重载需要注意，例如-> 和 * ，例子在下面
- 特殊的运算符重载需要注意，例如 = ，例子在下面
- 特殊的运算符重载需要注意，例如 () ，重载后相当于一个匿名函数，例子在下面
- 不要重载逻辑与&&和逻辑或|| ，原因是它们默认的特性是无法重载的，例如 false && (!false) 按照道理来讲直接计算左边的条件是false后右边的表达式是不会计算的，但是如果重载就会先将左边和右边的表达式提前计算出来放入重载函数的参数中，但是实际上有些时候左边的表达式确定结果后是不能执行右边的表达式，因为执行右边的表达式会报错


### 其实不一定需要使用运算符重载，内部的操作在类中实现函数一样可以做到，并且函数名称更让人容易区分

### 方式一：全局函数重载运算
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int a, b;

	Person() {}

	Person(int a,int b) {
		this->a = a;
		this->b = b;
	}
};

// operator关键字和需要重载的符号作为全局函数重载运算符
// 运算符的作用类型和运算符的作用分别是全局函数的参数类型和数量以及函数体的实现返回值决定
Person operator+(Person& p1,Person& p2) {
	Person p;
	p.a = p1.a+p2.a;
	p.b = p2.b + p2.b;
	return p;
}

int main() {
	Person p1(10, 10);
	Person p2(20, 30);

	// p1+p2是简写，其实是调用全局函数 operator+(p1, p2)
	Person p3 = p1 + p2;

	cout << p3.a << endl;
	cout << p3.b << endl;

	return 0;
}
~~~
---
 

### 方式二：类中函数重载运算符
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int a, b;

	Person() {}

	Person(int a,int b) {
		this->a = a;
		this->b = b;
	}
	
	// operator关键字和需要重载的符号作为全局函数重载运算符
    // 运算符的作用类型和运算符的作用分别是全局函数的参数类型和数量以及函数体的实现返回值决定
	Person operator+(Person& p1) {
		Person p;
		p.a = this->a + p1.a;
		p.b = this->b + p1.b;
		return p;
	}
};


int main() {
	Person p1(10, 10);
	Person p2(20, 30);

	// p1+p2是简写，其实是 p1.operator+(p2)
	Person p3 = p1 + p2;

	cout << p3.a << endl;
	cout << p3.b << endl;

	return 0;
}
~~~
---
 

### 同一运算符可以多次重载
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int a, b;

	Person() {}

	Person(int a,int b) {
		this->a = a;
		this->b = b;
	}

	// 同一运算符多次重载
	Person operator+(Person& p1) {
		Person p;
		p.a = this->a + p1.a;
		p.b = this->b + p1.b;
		return p;
	}

	// 同一运算符多次重载
	Person operator+(int a) {
		Person p;
		p.a = this->a + a;
		p.b = this->b;
		return p;
	}
};

int main() {
	Person p1(10, 10);

	// p1+p2是简写，其实是 p1.operator+(10)
	Person p3 = p1 + 10;

	cout << p3.a << endl;
	cout << p3.b << endl;

	return 0;
} 
~~~
---
 
### 特殊的运算符重载需要注意，例如++ 和 -- 在前面和后面时
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class Person {
public:
	int a;

	Person() {}

	Person(int a) {
		this->a = a;
	}

	// 前置++运算符重载
	int operator++() {
		return ++this->a;
	}

	// 后置++运算符重载，是使用匿名参数来区分前置++和后置++
	int operator++(int) {
		return this->a++;
	}
};

int main() {
	Person p1(10);
	// ++p1是简写，其实是 ++p1.a
	cout << ++p1 << endl;
	cout << p1.a << endl;

	Person p2(10);
	// p1++是简写，其实是 p2.a++
	cout << p2++ << endl;
	cout << p2.a << endl;

	return 0;
}
~~~
---
 

 

## 特殊的运算符重载需要注意，例如-> 和 * 

- ### 通过在类内部维护一个它自己在堆上的指针和重写析构函数使其在回收类对象在栈上的空间时同时回收内部指向堆上的对象，做到堆对象忘记回收时在方法栈中通过回收栈上的对象调用析构函数回收内部堆上的对象，但是这样访问内部堆上的对象会比较麻烦
- ### 解决内部堆对象访问麻烦可以使用内部的函数或者是重载类的运算符 -> 和 * 

~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std; 

class SmartPoint {
private:
	int age;

public:
	SmartPoint() {}

	SmartPoint(int age) {
		this->age = age;
	}

	// 智能指针：p指向堆上的对象
	SmartPoint(SmartPoint* p) {
		this->p = p;
	}

	// 当栈上的对象被回收时会调用析构函数，在析构函数中回收堆上对象
	~SmartPoint() {
		if (this->p) {
			delete this->p;
			this->p = NULL;
			cout << "栈上SmartPoint对象内部的堆上SmartPoint对象被回收了" << endl;
		}
	}

	int getAge() {
		return this->age;
	}

	void setAge(int age) {
		this->age=age;
	}

	// ->运算符重载，返回内部管理的堆对象指针
	SmartPoint* operator->() {
		return this->p;
	}

	// *运算符重载，返回内部管理的堆对象引用，注意返回值类型是引用，而不是一个新的对象
	SmartPoint& operator*() {
		return *(this->p);
	}

private:
	// 智能指针：p指向堆上的对象，通过栈上的对象操作堆上的对象，并且当栈上的对象被回收时会调用析构函数，在析构函数中回收堆上对象
	// 避免堆上的对象忘记回收造成内存泄漏
	SmartPoint* p;

};

int main() {

	SmartPoint sp(new SmartPoint(18));
	// 重载运算符-> 而 sp-> 得到的是内部堆对象的指针，其实通过sp->->getAge()调用函数 但是编译器会优化代码所以需要写成sp->getAge()
	cout << sp->getAge() << endl;

	sp->setAge(20);
	cout << sp->getAge() << endl;

	// 重载运算符* 而 *sp 得到的是内部堆对象的引用
	(*sp).setAge(30);
	cout << (*sp).getAge() << endl;
	
	return 0;
}
~~~
---
 

## 特殊的运算符重载需要注意，例如 = 

- ### 默认的operator=是浅拷贝函数，一旦类中出现有些属性存放在堆内存中，并且使用运算符 = 进行赋值，就会在方法栈结束时多次回收内部存放在堆内存中的属性而它们由于是浅拷贝占用同一内存空间，当第一次回收后，之后指向该空间的指针是野指针，而清空野指针指向的数据是危险操作，会引起谁也不知道的数据丢失
- ### 重写operator= 使其成为深拷贝函数可以避免上述问题

~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Person {
public:
	char* name;
public:
	Person() {
	}

	Person(const char* m_name, int m_age) {
		// 将name属性放入堆内存
		name = (char*)malloc(sizeof(m_name) + 1);
		strcpy(name, m_name);
	}

	// 默认的operator=是浅拷贝函数，重载运算符= 使其成为深拷贝，
	Person & operator=(const Person& p){
		// 将name属性放入堆内存
		name = (char*)malloc(sizeof(p.name) + 1);
		strcpy(name, p.name);
		return *this;
	}

	// 注意在析构函数中释放属性存放在堆内存空间
	// 如果使用浅拷贝会导致多次重复释放同一属性在堆中存放的内存空间
	// 其实第一次释放后该堆空间可能存放数据到该空间，
	// 而其他拷贝对象的属性还是指向已经释放堆空间而这些指针相当于是野指针，操作野指针清空指向的内存数据会发生未知的后果
	~Person() {
		cout << "析构函数被调用了" << endl;
		if (name != NULL) {
			free(name);
			name = NULL;
		}
	}
};

int main() {
	Person p("张三", 18);
	Person p2;
	Person p3;
	// 调用默认的opterot
	p3 = p2 = p;

	cout << p.name << endl;
	cout << p2.name << endl;
	cout << p3.name << endl;
	return 0;
}
~~~
---
 

## 特殊的运算符重载需要注意，例如 () ，重载后相当于一个匿名函数，例子如下
 
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

class Person {
public:
	int a;
public:
	Person(int a) {
		this->a = a;
	}

    // operator重载() 使其成为匿名函数
	int operator()(){
		return this->a;
	}
	
};

int main() {
	Person p(18);

	cout << p() << endl;
	
	return 0;
}
~~~
---
 
 