# 服务端代码
~~~go
package main

import (
	"bytes"
	"fmt"
	"net"
	"strings"
)

func main() {

	// 创建socket套接字，监听当前主机的8080端口，0.0.0.0表示的是本机，无论是IPv4还是IPv6
	listen, err := net.Listen("tcp", "0.0.0.0:8080")
	if err != nil {
		fmt.Printf("创建socket套接字监听端口失败，错误是%v\n", err)
	}

	defer listen.Close()

	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Printf("socket套接字与客户端建立连接失败，错误是%v\n", err)
		}

		go handleClientMessage(conn)
	}
}

func handleClientMessage(conn net.Conn) {
	fmt.Printf("与客户端的连接建立成功，客户端的IP地址和端口是：%+v\n", conn.RemoteAddr().String())
	for {
		s := make([]byte, 1024)
		n, readErr := conn.Read(s)
		if n == 0 {
			continue
		}
		if readErr != nil {
			fmt.Printf("读取客户端传入的数据出错，错误是%v\n", readErr)
			break
		}

		fmt.Printf("客户端输入的字符串是：%v", string(s[:n]))
		if strings.Trim(string(s[:n]), "\r\n") == "exit" {
			break
		}

		_, writeErr := conn.Write(bytes.ToUpper(s))
		if writeErr != nil {
			fmt.Printf("发生给客户端转换为大写的字符数组出错，错误是%v\n", writeErr)
			break
		}
	}
	fmt.Printf("与客户端断开了连接，客户端的IP地址和端口是：%+v\n", conn.RemoteAddr().String())
	conn.Close()
}
~~~

# 客户端代码
~~~go
package main

import (
	"bufio"
	"fmt"
	"net"
	"os"
)

func main() {
	// 连接服务端的socket，
	conn, err := net.Dial("tcp", "127.0.0.1:8080")
	if err != nil {
		fmt.Printf("连接服务端的socket套接字失败，错误是%v\n", err)
	}

	defer conn.Close()

	reader := bufio.NewReader(os.Stdin)
	for true {
		fmt.Println("请输入需要转换的大写字符串，输入exit会退出程序")
		readLine, _ := reader.ReadString('\n')

		_, writeErr := conn.Write([]byte(readLine))
		if writeErr != nil {
			fmt.Printf("发生客户端数据时出错，错误是%v\n", writeErr)
			break
		}

		if readLine == "exit\r\n" {
			break
		}

		s := make([]byte, 1024)
		n, readErr := conn.Read(s)
		if readErr != nil {
			fmt.Printf("读取服务端返回的数据出错，错误是%v\n", readErr)
			break
		}
		fmt.Printf("服务端的返回的大写字符串是：%v", string(s[:n]))
	}
	
	fmt.Println("与服务端断开了连接")
}
~~~
