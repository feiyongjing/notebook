# c++的IO主要是

- 系统的标准设备输入输出，例如键盘的输入和显示屏的输出，这些统称标准IO流，iostream就是标准输入输出的类库
- 磁盘设备文件存储的输入输出统称文件IO流
- 内存空间一般是以一个字符数组作为存储空间，对于这种存储的输入输出被称为串IO流
 

## 标准输入读取（cin是标准输入流对象）

### cin.get()从标准输入流中读取一个字符
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	string a = "";

	// cin.get()读取一个标准输入的字符
	// 以下是读取一行字符串
	char c = cin.get();
	while (c != '\n') {
		a.append(&c,sizeof(c));
		c = cin.get();
	}

	cout << "读取的字符串"<<a << endl;
	return 0;
}
~~~
---

### cin.get(_Elem* _Str, streamsize _Count) 读取指定数量的字符直到出现结束字符 '\n' ，读取的字符写入指定内存地址中
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char a[1024] = {'0'};

	// cin.get(_Elem* _Str, streamsize _Count) 
	// 从流中读取指定数量的字符到指定的开始地址中，注意读取一行字符串直到出现结束字符 '\n'
	cin.get(a,1024);
	cout << "读取的字符串"<<a << endl;
	
	if (cin.get()=='\n') {
		cout << "读取一行字符串时末尾的换行符没有读取，而是遗留在缓存区中" << endl;
	}else{
		cout << "读取一行字符串时末尾的换行符读取了" << endl;
	}
	return 0;
}
~~~
---

### getline(_Elem* _Str, streamsize _Count) 读取指定数量的字符直到出现结束字符 '\n' ，读取的字符写入指定内存地址中
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char a[1024] = {'0'};

	// getline(_Elem* _Str, streamsize _Count)
	// 从流中读取指定数量的字符到指定的开始地址中，注意读取一行字符串直到出现结束字符 '\n'
	cin.getline(a,1024);
	cout << "读取的字符串" << a << endl;
	cout << "读取一行字符串时末尾的换行符没有读取，也不在缓存区中，而是从缓冲区读取一行字符串时刷新缓冲区丢弃了没有读取的换行符" << endl;
	return 0;
}
~~~
---

### ignore(streamsize _Count = 1)忽略缓冲区的字符，默认是忽略一个，可以指定忽略指定数量的字符
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char a[1024] = {'0'};

	// ignore(streamsize _Count = 1)
	// 忽略缓冲区的字符，默认是忽略一个，可以指定忽略指定数量的字符
	cin.ignore(3);
	cin.getline(a, 1024);
	cout << "读取的字符串开头少3个字符" << a << endl;
	return 0;
}
~~~
---

### peek() 查看缓冲区的头字符，但是与get()的区别是没有从缓冲区取走头字符，头字符还是在缓冲区中
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char a[1024] = {'0'};

	// peek() 查看缓冲区的头字符，但是与get()的区别是没有从缓冲区取走头字符，头字符还是在缓冲区中
	char c=cin.peek();
	cin.getline(a, 1024);
	cout << "读取的头字符" << c << endl;
	cout << "读取的字符串" << a << endl;
	return 0;
}
~~~
---

### putback(_Elem _Ch)放入一个字符到缓冲区的头部位置
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char a[1024] = {'0'};

	// putback(_Elem _Ch) 
	// 放入一个字符到缓冲区的头部位置
	cin.putback('x');
	cin.getline(a, 1024);
	cout << "读取的字符串会在前面多一个x字符" << a << endl;
	return 0;
}
~~~
---

## 标准输出（cout是标准输出流对象）

### cout.flush();  是刷新缓冲区，将缓冲区的数据输出到输出设备上，注意该方法只有linux下有效
 
### put(_Elem _Ch) 向缓冲区写入一个字符
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
using namespace std;

int main() {
	cout << "向输出流写入字符串：";
	// put(_Elem _Ch) 向输出流写入一个字符
	// 可以链式调用
	cout.put('h').put('e').put('l').put('l').put('o');
	return 0;
}
~~~
---

### write(const _Elem* _Str, streamsize _Count) 输出流写入指定数量的字符
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
using namespace std;

int main() {
	char buf[] = "hello world";
	cout << "输出流写入字符串：";
	// write(const _Elem* _Str, streamsize _Count) 输出流写入指定数量的字符
	cout.write(buf,sizeof(buf));
	return 0;
}
~~~
---

### 格式化标准输出函数
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <iomanip> // 标准输入输出格式化库
#include <string> 
using namespace std;

// 标准输出格式化函数
void fun1() {
	int number = 99;
	cout.width(20);           // 占据输出流20个字符，实际字符不到20也占据20个字符多余空间默认使用空格占用
	cout.fill('*');           // 多余的空间使用指定字符填充
	cout.setf(ios::left);     // 实际数据使用左对齐，默认是右对齐
	cout.unsetf(ios::dec);    // 卸载十进制
	cout.setf(ios::oct);      // 安装8禁止
	cout.setf(ios::showbase); // 显示数据格式：10进制数不显示，8进制数开头显示0，16进制数开头显示0x
	cout.unsetf(ios::oct);    // 卸载8进制
	cout.setf(ios::hex);      // 安装16进制
	cout<< number;
}

// 以下链式函数进行标准输出格式化需要引入 #include <iomanip> 标准输入输出格式化库
void fun2() {
	int number = 49;
	cout<< setw(20)                      // 占据输出流20个字符，实际字符不到20也占据20个字符多余空间默认使用空格占用
		<< setfill('*')                  // 多余的空间使用指定字符填充
		<< setiosflags(ios::left)        // 实际数据使用左对齐，默认是右对齐
		<< setiosflags(ios::showbase)    // 显示数据格式：10进制数不显示，8进制数开头显示0，16进制数开头显示0x
		<< hex                           // 安装16进制
		<< number;
}

int main() {
	fun1();
	fun2();
	return 0;
}
~~~
---
 
## 文件IO流操作
~~~c++
# define _CRT_SECURE_NO_WARNINGS
#include <iostream> // 标准输入输出
#include <string> 
#include <fstream>  // 文件IO流
using namespace std;

// 文件的打开
void wirte() {

	// 文件的打开方式有如下一些
	// ios::in                 以只读方式打开文件
	// ios::out                以写方式（这是默认的方式），如果文件已经存在会将原有内容全部清除
	// ios::app                以写方式打开文件，写入的数据添加在文件内容的末尾
	// ios::ate                打开一个已经存在的文件，文件的指针指向文件末尾
	// ios::trunc              打开一个文件如果文件已经存在则删除其中的全部数据，如果文件不存在，则建立新的文件
	// ios::binary             二进制方式大打开文件，如果不指定编码格式默认是ASCLL方式编码打开
	// ios::nocreate           打开一个文件。如果文件不存在则打开失败不会创建新的文件
	// ios::noreplace          如果文件不存在就创建新的文件，如果文件已经存在则操作失败
	// ios::in | io::out       以可读可写的方式打开文件
	// ios::binary | ios::in   以二进制的方式打开文件读取内容
	// ios::binary | ios::out  以二进制的方式打开文件写入内容
	
	// 获取文件的ofstream可以写入文件内容
	//ofstream ofs("./test.txt", ios::out| ios::trunc);  构造器设置打开的文件路径和打开方式

	ofstream ofs;
	ofs.open("./test.txt", ios::out | ios::trunc); // 通过open手动设置文件打开路径和打开方式

	if (!ofs.is_open()) {
		throw "文件打开失败";
	}

	ofs << "姓名：孙悟空" << endl;
	ofs << "年龄：999" << endl;
	ofs << "性别：男" << endl;

	// 关闭文件
	ofs.close();
}

void read() {

	// 获取文件的ofstream可以读取文件内容
	ifstream ifs("./test.txt", ios::in); 

	if (!ifs.is_open()) {
		throw "文件打开失败";
	}

	char buf[1024] = {0};

	// 第一种读取文件内容方式
	while (ifs >> buf) {
		cout << buf << endl;
	}

	// 第二种读取文件内容方式
	/*while (ifs.getline(buf, sizeof(buf))) {
		cout << buf << endl;
	}*/

	// 第三种读取文件内容方式
	/*string str;
	while (getline(ifs, str)) {
		cout << str << endl;
	}*/

	// 第四种读取文件内容方式
	/*char c;
	while ( (c = ifs.get()) != EOF) {
		cout << c;
	}*/

	// 关闭文件
	ifs.close();
}

int main() {
	wirte();
	read();
	return 0;
}
~~~
---
