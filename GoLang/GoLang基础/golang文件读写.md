### 通过缓冲区流式读文件，适用于大文件读取
~~~go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	//os.Open(”./test.txt“)  Open是只读方式打开文件，如果需要读写方式打开请使用 OpenFile

	// 第一个参数指定文件路径
	// 第二个参数指定文件打开方式，注意一般情况下不要使用 os.O_TRUNC 打开文件，使用该模式打开文件会清空原因的文件内容
	// 第三个参数是文件不存在时创建文件并赋予的权限，如果文件已经存在一般是传 0。windows下该参数不起作用
	file, err := os.OpenFile("./test.txt", os.O_RDWR|os.O_CREATE, 0)

	if err != nil {
		panic("文件打开失败，打印问题并退出程序")
	}

	// 延迟调用文件关闭
	defer file.Close()

	// 创建缓冲区，使用流式读取文件内容，每次将文件读取一部分放入缓冲区
	// 缓冲区的默认大小是4096，如果需要设置缓冲区大小请使用 NewReaderSize 函数来创建缓冲区并设置初始大小
	reader := bufio.NewReader(file)

	for {
		// 读取到指定字符停止，例如换行符，即一行一行的读取
		lineString, err := reader.ReadString('\n')
		if err == io.EOF { // 判断文件内容是否读取到文件末尾
			fmt.Printf(lineString) // 可能最后一行没有换行符而是直接结束
			break
		} else if err != nil {
			panic("文件内容读取失败，打印问题并退出程序")
		} else {
			fmt.Printf(lineString)
		}
	}
}
~~~

### 一次性读取文件内容，适用于比较小的文件
~~~go
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
    // 使用ioutil.ReadFile 读取文件无需手动关闭文件，该函数已经处理好了文件的关闭，返回的是文件的byte数组
	bytes, err := ioutil.ReadFile("./test.txt")

	if err != nil {
		panic("文件读取失败，打印问题并退出程序")
	}

	fmt.Println(string(bytes))
}
~~~

### 常规写入内容到文件末尾
~~~go
package main

import (
	"os"
)

func main() {

	// 第一个参数指定文件路径
	// 第二个参数指定文件打开方式，注意一般情况下不要使用 os.O_TRUNC 打开文件，使用该模式打开文件会清空原因的文件内容
	// 第三个参数是文件不存在时创建文件并赋予的权限，如果文件已经存在一般是传 0。windows下该参数不起作用
	file, err := os.OpenFile("./test.txt", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0)

	if err != nil {
		panic("文件打开失败，打印问题并退出程序")
	}

	// 延迟调用文件关闭
	defer file.Close()

	// 字节方式写入，返回写入的字节数量
	_, err = file.Write([]byte("write : 哈哈哈\n"))
	if err != nil {
		panic(err)
	}
	
	// 字符串写入，返回写入的字节数量
	_, err = file.WriteString("writeString : 哈哈哈\n")
	if err != nil {
		panic(err)
	}
}
~~~

### 通过缓冲区流式写入文件，适用于写入大量内容
~~~go
package main

import (
	"bufio"
	"os"
)

func main() {
	// 第一个参数指定文件路径
	// 第二个参数指定文件打开方式，注意一般情况下不要使用 os.O_TRUNC 打开文件，使用该模式打开文件会清空原因的文件内容
	// 第三个参数是文件不存在时创建文件并赋予的权限，如果文件已经存在一般是传 0。windows下该参数不起作用
	file, err := os.OpenFile("./test.txt", os.O_RDWR|os.O_CREATE|os.O_APPEND, 0)

	if err != nil {
		panic("文件打开失败，打印问题并退出程序")
	}

	// 延迟调用文件关闭
	defer file.Close()

	// NewWriter 默认缓冲区大小是 4096
	// 需要使用自定义缓冲区的writer，请使用 NewWriterSize()方法
	buf := bufio.NewWriter(file)

	// 字节写入
	_, err = buf.Write([]byte("buffer Write : 哈哈哈"))
	if err != nil {
		panic(err)
	}

	// 字符串写入
	_, err = buf.WriteString("buffer WriteString : 哈哈哈")
	if err != nil {
		panic(err)
	}

	// 将缓冲中的数据刷新写入文件
	err = buf.Flush()
	if err != nil {
		panic(err)
	}
}
~~~

### 判断文件或目录是否存在
~~~go
package main

import "os"

func main() {
	exists, _ := PathExists("./test.txt")
	println(exists)
}

func PathExists(path string) (bool, error) {
	// 查看文件信息
	_, err := os.Stat(path)

	if err == nil {
		return true, err
	} else if os.IsNotExist(err) {
		return false, err
	}
	return true, err
}
~~~

### 拷贝文件
~~~go
package main

import (
	"io"
	"os"
)

func main() {
	copySize, _ := copyFile("./test.txt", "./test1.txt")
	println(copySize)
}

func copyFile(srcFile, destFile string) (int64, error) {
	file1, err := os.Open(srcFile)
	if err != nil {
		return 0, err
	}
	file2, err := os.OpenFile(destFile, os.O_WRONLY|os.O_CREATE, os.ModePerm)
	if err != nil {
		return 0, err
	}
	defer file1.Close()
	defer file2.Close()

	// io包中私有函数copyBuffer() 默认缓冲区是32K
	// 使用 CopyBuffer 可以手动设置缓冲区大小
	// 返回成功拷贝的字节数和错误
	return io.Copy(file2, file1)
}
~~~
