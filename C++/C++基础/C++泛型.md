# 泛型：类型参数化，表示特定的类型，所以使用这些类型的地方都必须保证一致

### 泛型的实现：本质是进行两次编译，
- 第一次编译是源代码编译成可执行程序时编译检查泛型使用是否有语法错误，以及调用泛型代码时上下文类型是否一致
- 第二次编译是真正运行代码时根据具体使用的类型替换掉泛型占据的类型，然后进行编译代码，再运行代码

### 泛型使用注意事项
- 泛型不会使用默认的隐式类型转换，如果有需要请声明
- 泛型函数与泛型函数、泛型函数与普通函数是允许发生函数的重载的
- 泛型函数和普通函数都匹配一个函数调用时，实际会调用普通函数，如果强制调用泛型函数则调用时不写泛型的具体类型，例如： 函数名<>();

### 常见的泛型函数
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;


// 泛型模板，表示特定的类型，所以使用这些类型的地方都必须保证一致
// 写法template <class T> 或者是template <typename T> , 其实class和typename都是一样的
template <class T> int fun1(T a,T b) {
	return a + b;
};

int main() {
	// 调用泛型函数时必须声明泛型的具体类型
	cout << fun1<int>(10, 23) << endl;

	return 0;
}

~~~

### 泛型类
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

template <class T,class L> class Person {
public:
	T name;
	L age;
	
	Person(T name,L age) {
		this->name = name;
		this->age = age;
	}
};

int main() {
	// 泛型类创建对象
	Person<string> p("张三", 18);

	cout << p.name << endl;
	cout << p.age << endl;

	return 0;
}
~~~
---
 
### 继承泛型类的写法和类中的泛型函数在类外实现
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

template <class T> class Animal {
public:
	T name;
	Animal(T name) {
		this->name = name;
	}

	void fun1(T name);
};

// 泛型函数写在类外时，必须包容类的泛型模板
template <class T> void Animal<T>::fun1(T name) {
	cout << name << endl;
}

// 子类继承父类时，如果父类使用了泛型模板，则子类必须包容父类的泛型模板
template <class T> class Person : public Animal<T> {
public:
	// 调用父类的泛型函数时必须声明泛型的具体类型
	Person(T name):Animal<T>(name) {}
};

int main() {
	// 泛型类创建对象
	Person<string> p("张三");

	cout << p.name << endl;

	p.fun1("泛型函数写在类外");
	return 0;
}

~~~
---
 