### 结构体与类的相同点是都有属性成员和函数，不同点是结构体中的成员和函数都没有权限控制（默认是public）,而类中的成员和函数有权限控制

# 结构体
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string>   // 字符串引入
using namespace std; 

// C++中一个没有成员属性的结构体创建出来的结构体占用一个字节
struct Student {
	int id;
	string name;
	int age;
	char sex[5];
	char tele[20];
	float score;

	string getName() {
		return name;
	}
};

int main() {
	struct Student s= { 1,"张三" };
	cout << s.getName() << endl;
	return 0;
}
~~~
---
 
# 类
~~~c++
# define _CRT_SECURE_NO_WARNINGS 
 
#include <iostream> // 标准输入输出 
# define PI 3.24 
 
using namespace std; 
 
// C++中一个没有成员属性的类创建出来的对象占用一个字节
// 类中的静态成员属性和静态成员函数、非静态成员函数不属于对象，所以对象的大小不计算这些
// public是公共权限访问，类外可以访问
// private是私有权限(只有当前类)访问
// protected只有当前类和子类有权限访问 
class Circle { 
public: 
    int m_r; 
 
    double calculateZC() { 
		a = 10;
		b = 20;
		c = 30;
		cout << a << endl;
		cout << b << endl;
		cout << c << endl;
        return 2 * PI * m_r; 
    } 
private:
    int a;
protected:
    int b;
    int c;
public:
    int d;
}; 
 
int main() {
	Circle c; // 根据类创建一个圆，也就是实例化对象 
	c.m_r = 10; // 设置圆的半径 
	c.d = 100;
	cout << c.d << endl;
	cout << "圆的周长是：" << c.calculateZC() << endl;
	return 0;
}
~~~
---

 

 